# Sharding support in {{pcsm.full_name}} (Technical Preview)

!!! warning "Technical Preview"

    Sharding support is available starting with {{pcsm.full_name}} 0.7.0 and is currently in technical preview stage. We encourage you to try it out and share your feedback. This will help us improve the feature in future releases.

{{pcsm.full_name}} supports replication between sharded MongoDB clusters, enabling you to migrate or synchronize data from one sharded deployment to another. This capability allows you to migrate sharded clusters with minimal downtime and synchronize data between sharded clusters for testing or development purposes.

## Overview

The workflow for sharded clusters is similar to replica sets. See [How {{pcsm.full_name}} works](intro.md#replication-workflows) for the complete workflow overview. The key difference is that {{pcsm.short}} connects to `mongos` instances on both the source and target clusters instead of replica set members.

Since {{pcsm.short}} connects through `mongos`, the cluster topology doesn't matter. This means the source and target clusters can have different numbers of shards.

Also, {{pcsm.short}} replicates data and not metadata. This means chunk distribution as well as the primary shard name for a collection may differ on source and target clusters.

## Prerequisites

* {{pcsm.full_name}} version 0.7.0 or later
* Source and target clusters must be sharded MongoDB deployments
* Both clusters must be running the same MongoDB version. Check [Version requirements](deployment.md#version-requirements) for more information about supported versions.

## Connection string format

When connecting to sharded clusters, use the standard MongoDB connection string format but specify `mongos` hostname and port instead of replica set members:

```{.text .no-copy}
mongodb://user:pwd@mongos-host:port/[authdb]?[options]
```

Since {{pcsm.short}} connects through `mongos`, you don't need to specify individual shard members or config servers in the connection string. The `mongos` router handles routing to the appropriate shards.

For detailed information about authentication and connection string configuration, see [Configure authentication in MongoDB](install/authentication.md).

## Sharding-specific behavior

### Initial sync preparation

Before starting the initial sync, {{pcsm.short}} checks which collections are sharded on the source cluster and creates corresponding sharded collections on the destination cluster. The sharding key is preserved from the source cluster. For ranged shard keys, {{pcsm.short}} also copies the initial chunk boundaries and ownership to the target; it does not replicate sharding metadata afterwards.

For a ranged shard key, immediately after it shards a collection on the target and before copying any documents, {{pcsm.short}} pre-splits the collection using the source chunk boundaries. Hashed collections retain the layout created by `shardCollection`. See [Chunk distribution](#chunk-distribution).

### Balancer operation

{{pcsm.full_name}} connects to source and target clusters via a `mongos` instance. Therefore, you do not need to disable the balancer on either the source or target cluster before starting replication. The target cluster's balancer continues to operate normally and manages chunk distribution according to its own sharding configuration and balancer settings.

The target starts from the same chunk boundaries as the source, so a chunk migration on the source arrives where the target expects it. That makes it safe to leave the balancer running during the sync, which matters in write-heavy clusters where turning it off is not an option. See [Manage sharded cluster balancer :octicons-link-external-16:](https://www.mongodb.com/docs/manual/tutorial/manage-sharded-cluster-balancer/){:target="_blank"} in the MongoDB documentation.

## Chunk distribution

!!! admonition "Version added: 0.10.0"

When MongoDB shards an empty collection on a ranged shard key, it creates a single chunk covering the entire range of shard key values. See [Data partitioning with chunks :octicons-link-external-16:](https://www.mongodb.com/docs/manual/core/sharding-data-partitioning/){:target="_blank"} in the MongoDB documentation. A clone into that collection would therefore write to a single shard, and the target balancer would move the data afterwards.

{{pcsm.short}} therefore recreates the source chunk boundaries on the target before copying any documents, so the clone writes to every shard from the start and no rebalancing wave follows. Matching boundaries are also what makes it safe to leave the balancer running on the source, as described in [Balancer operation](#balancer-operation).

This runs automatically for every sharded collection, immediately after {{pcsm.short}} shards it on the target. There is no flag and nothing to configure.

| **Source collection** | **Target shards** | **Result on the target** |
|-----------------------|-------------------|--------------------------|
| Hashed shard key | Any number | The layout that `shardCollection` creates, unchanged. |
| Ranged shard key | Same number as the source | The same chunk boundaries and the same ownership pattern as the source. |
| Ranged shard key | Different number from the source | The same chunk boundaries, with each target shard holding roughly the same volume of data. |

!!! note "The layout is a starting point, not a copy"

    {{pcsm.short}} reads the source boundaries once, before the clone, and does not replicate sharding metadata afterwards. Later migrations, splits, merges, and resharding on the source have no effect on the target, so the two layouts drift apart as the balancers work. That is expected and does not indicate a replication problem.

### Hashed shard keys

{{pcsm.short}} does not pre-split hashed collections, and does not need to. MongoDB already spreads the initial chunks evenly across the shards for a hashed shard key, so {{pcsm.short}} keeps that layout. See [Hashed sharding :octicons-link-external-16:](https://www.mongodb.com/docs/manual/core/hashed-sharding/){:target="_blank"} in the MongoDB documentation.

The number of chunks depends on your MongoDB version. With three target shards, MongoDB 6.0 and 7.0 create six chunks and MongoDB 8.0 creates three.

### Ranged shard keys

With the same number of shards on both sides, the target gets the source boundaries and the same ownership pattern. Shards are paired in sorted order, so a range does not necessarily land on the target shard whose name resembles its source shard.

With different shard counts, the boundaries still come from the source, but the largest chunks are placed first, each on whichever target shard holds the least data at that point. Every shard ends up owning chunks and holding roughly the same volume. The estimate carries across collections, so the large chunks of several collections do not all collect on one shard.

??? example "How the two cases look"

    ```{.text .no-copy}
    Same number of shards
    ---------------------
    Source: [MinKey, 100) -> src-a    Target: [MinKey, 100) -> tgt-a
            [100, MaxKey) -> src-b            [100, MaxKey) -> tgt-b

    Different number of shards
    --------------------------
    Target shards: tgt-a, tgt-b
    Source chunk sizes: 100 MB, 60 MB, 40 MB

    100 MB -> tgt-a        Estimated result:
     60 MB -> tgt-b        tgt-a: 100 MB
     40 MB -> tgt-b        tgt-b: 100 MB
    ```

### If the pre-split fails

A failed pre-split fails the clone for that instance, and there is no fallback to loading into an unsplit collection. Check the log for the reported failure, resolve it on the target cluster, then restart replication with `pcsm resume --from-failure`. See [Resume the replication](install/usage.md#resume-the-replication), [Logging in {{pcsm.full_name}}](logging.md), and the [Troubleshooting guide](troubleshooting.md).

### Check the layout on the target

Connect to the target `mongos` and look at how a replicated collection is spread:

```javascript
db.getSiblingDB('<database>').<collection>.getShardDistribution()
```

Look for data on every shard rather than an exact match with the source, since counts differ even immediately after the clone and keep changing as the balancer works. For chunk counts per shard across the cluster, use [sh.status() :octicons-link-external-16:](https://www.mongodb.com/docs/manual/reference/method/sh.status/){:target="_blank"}.

## Usage

The commands and API endpoints for sharded cluster replication are the same as for replica set replication. The workflow follows the same stages as replica set replication. See [How {{pcsm.full_name}} works](intro.md#replication-workflows) for the complete workflow overview and [Use {{pcsm.full_name}}](install/usage.md) for detailed command instructions.

## Next steps

* [Install {{pcsm.full_name}}](installation.md)
* [Configure authentication](install/authentication.md)
* [Start replication](install/usage.md)
* [Monitor replication status](install/usage.md#check-the-replication-status)
* [Monitor PCSM performance with Percona Monitoring and Management](pmm-setup.md)
