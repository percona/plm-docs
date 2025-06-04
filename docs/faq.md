# FAQ

## What are the typical use cases for {{pml.full_name}}?

{{pml.full_name}} is a data migration tool that you can use for various use cases. Among them are:

- Migration from MongoDB Atlas or MongoDB Enterprise to Percona Server for MongoDB.
- Cluster synchronization across environments, such as staging to production.
- Setup of live replication for backup, testing, or audit environments.
- Downtime minimization during a migration by using continuous sync until cutover.

## Which MongoDB versions are supported?

- Source clusters: MongoDB 6.0.17 and later, including Atlas and MongoDB Enterprise editions.
- Target clusters: Percona Server for MongoDB 6.0.17 and later.

Check [Supported deployments](deployment.md) to learn more.

## Can I sync from Atlas to a self-hosted Percona Server for MongoDB?

Yes. {{pml.full_name}} is explicitly built to support Atlas to Percona Software migrations with minimal effort.

## Does {{pml.full_name}} require a replica set on the source or target?

Yes. Both the source and target must be replica sets. Sharded clusters are not yet supported, but planned.

## Does {{pml.full_name}} support bidirectional sync?

No. {{pml.full_name}} currently supports one-way synchronization only (source → target). However, you can re-run Percona {{pml.full_name}} with a reversed connection strings to do the other direction sync.

## Is there a way to monitor sync progress?

Yes. {{pml.full_name}} provides Prometheus metrics exposed at the `/metrics` endpoint and provides detailed logging for sync status, lag, and errors. See how you can [configure monitoring with PMM](pmm-setup.md). You can also use a monitoring tool of your choice.

## Can I filter which databases or collections to sync?

Yes. {{pml.full_name}} allows you to include/exclude filters for specific databases or collections. This option is currently available via HTTP API. The command-line support will be added in future releases.

## How does {{pml.full_name}} handle failures?

{{pml.full_name}} is designed with resilience in mind:

- It supports automatic retries on transient errors.
- It uses checkpointing to resume from the last known sync point after restart.
- Logs include detailed error reporting for troubleshooting.

## What features are planned for future releases?

Sharded cluster support and high availability are the next features we are working on.   
