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

Before starting the initial sync, {{pcsm.short}} checks which collections are sharded on the source cluster and creates corresponding sharded collections on the destination cluster. The only sharding configuration preserved from the source cluster is the sharding key; all other sharding details are handled internally by the destination cluster.

Immediately after it shards a collection on the target, and before it copies any documents into it, {{pcsm.short}} pre-splits that collection so that the clone writes spread across all target shards. See Chunk distribution.

### Balancer operation

{{pcsm.full_name}} connects to source and target clusters via a `mongos` instance. Therefore, you do not need to disable the balancer on either the source or target cluster before starting replication. The target cluster's balancer continues to operate normally and manages chunk distribution according to its own sharding configuration and balancer settings.

The target starts from the same chunk boundaries as the source, so a chunk migration on the source arrives where the target expects it. That makes it safe to leave the balancer running during the sync, which matters in write-heavy clusters where turning it off is not an option. See [Manage sharded cluster balancer :octicons-link-external-16:](https://www.mongodb.com/docs/manual/tutorial/manage-sharded-cluster-balancer/){:target="_blank"} in the MongoDB documentation.

## Chunk distribution

!!! admonition "Version added: 0.10.0"

Sharding an empty collection on a ranged shard key gives you a single chunk that covers the whole key range, and the balancer only starts spreading data once documents arrive. For a clone, that is the worst possible starting point. Every document {{pcsm.short}} writes lands on one shard, that shard absorbs the entire write load, and when the clone finishes the balancer begins a long migration of data that never needed to be in one place.

{{pcsm.short}} pre-splits the target collection before it copies anything. It reads the chunk layout of the source collection, recreates those boundaries on the target, and places the resulting chunks across the target shards. Clone writes then spread across every shard from the first document, and no rebalancing wave follows the clone.

This happens automatically. There is no flag to set, nothing to enable, and no way to turn it off. {{pcsm.short}} identifies the source collection by UUID, keeps the chunk boundaries in order, and applies them with the standard MongoDB sharding commands.

| **Source collection** | **Target shards** | **What {{pcsm.short}} does** |
|-----------------------|-------------------|------------------------------|
| Hashed shard key | Any number | Nothing. The target keeps the layout that `shardCollection` creates. |
| Ranged shard key | Same number as the source | Mirrors the source chunk boundaries and their ownership pattern. |
| Ranged shard key | Different number from the source | Replays the source boundaries and places the chunks so that each target shard holds roughly the same volume of data. |

!!! note "The layout is a starting point, not a copy"

    {{pcsm.short}} reads the source chunk boundaries once, before the clone. It does not replicate sharding metadata afterwards, so later chunk migrations, splits, merges, and resharding on the source have no effect on the target layout. The two clusters drift apart as soon as either balancer moves data. A layout that no longer matches the source is expected and does not indicate a replication problem.


### Hashed shard keys

{{pcsm.short}} does no pre-splitting for hashed shard keys, and that is deliberate. The `shardCollection` command already produces an evenly distributed layout across all shards on every supported MongoDB version, so there is nothing to improve and {{pcsm.short}} keeps what MongoDB created.

The number of initial chunks depends on the server version. With three target shards:

* MongoDB 6.0 and 7.0 create six chunks, roughly two per shard.
* MongoDB 8.0 creates three chunks, roughly one per shard.

See [Hashed sharding :octicons-link-external-16:](https://www.mongodb.com/docs/manual/core/hashed-sharding/){:target="_blank"} in the MongoDB documentation for how the server builds that initial layout.

### Ranged shard keys with the same number of shards

When both clusters have the same number of shards, {{pcsm.short}} reproduces the source layout directly. It sorts the shard IDs on each side, pairs them by position, replays every source chunk boundary on the target, and puts each target chunk on the shard paired with its source owner.

```{.text .no-copy}
Source shards: src-a, src-b
Target shards: tgt-a, tgt-b

Source layout:
[MinKey, 100)  ->  src-a
[100, MaxKey)  ->  src-b

Target layout:
[MinKey, 100)  ->  tgt-a
[100, MaxKey)  ->  tgt-b
```

!!! note "Shard names are paired, not matched"

    Pairing is by sorted position, so the shard that owns a range on the target is not necessarily the one with a similar name on the source. What {{pcsm.short}} reproduces is the shape of the distribution, not the shard names.

### Ranged shard keys with a different number of shards

Source ownership cannot be mirrored when the shard counts differ, so {{pcsm.short}} aims for even data volume instead. It estimates the size of every source chunk, works through the chunks from largest to smallest, and assigns each one to the target shard holding the least estimated data so far. It then replays the source boundaries and places the chunks according to those assignments.

```{.text .no-copy}
Target shards: tgt-a, tgt-b
Source chunk sizes: 100 MB, 60 MB, 40 MB

100 MB  ->  tgt-a
 60 MB  ->  tgt-b
 40 MB  ->  tgt-b

Estimated result:
tgt-a: 100 MB
tgt-b: 100 MB
```

The running size estimate carries across collections rather than resetting for each one, so a large chunk from one collection and a large chunk from the next do not both land on the same target shard. The source boundaries are preserved either way. Only the ownership changes.

### If the pre-split fails

A failed pre-split fails the clone for that instance. {{pcsm.short}} does not fall back to loading into an unsplit collection, because that would quietly reintroduce the single-shard bottleneck the pre-split exists to prevent.

Fix the underlying problem on the target cluster, then restart replication with `pcsm resume --from-failure`. See [Resume the replication](install/usage.md#resume-the-replication), [Logging in {{pcsm.full_name}}](logging.md), and the [Troubleshooting guide](troubleshooting.md).

### Check the layout on the target

Connect to the target `mongos` and look at how a replicated collection is spread:

```javascript
db.getSiblingDB('<database>').<collection>.getShardDistribution()
```

For chunk counts per shard across the cluster, use `sh.status()`. See [db.collection.getShardDistribution() :octicons-link-external-16:](https://www.mongodb.com/docs/manual/reference/method/db.collection.getShardDistribution/){:target="_blank"} and [sh.status() :octicons-link-external-16:](https://www.mongodb.com/docs/manual/reference/method/sh.status/){:target="_blank"} in the MongoDB documentation.

Look for data on every shard rather than an exact match with the source. Chunk counts and document counts per shard differ from the source even immediately after the clone, and they keep changing as the balancer works.

## Usage

The commands and API endpoints for sharded cluster replication are the same as for replica set replication. The workflow follows the same stages as replica set replication. See [How {{pcsm.full_name}} works](intro.md#replication-workflows) for the complete workflow overview and [Use {{pcsm.full_name}}](install/usage.md) for detailed command instructions.

## Next steps

* [Install {{pcsm.full_name}}](installation.md)
* [Configure authentication](install/authentication.md)
* [Start replication](install/usage.md)
* [Monitor replication status](install/usage.md#check-the-replication-status)
* [Monitor PCSM performance with Percona Monitoring and Management](pmm-setup.md)
