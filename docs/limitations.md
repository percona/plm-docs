---
title: Known issues and limitations
author: Radoslaw Szulgo
---
# Known issues and limitations

This page lists known limitations for using Percona MongoLink

## Versions and topology

* Sharded clusters support
* EOLed MongoDB version
* **Percona MongoLink** supports only Replica Set to Replica Set clusters synchronization
* You can synchronize Percona Server for MongoDB or MongoDB Community/Enterprise Advanced/Atlas within the compatible versions - 6.0 to 6.0, 7.0 to 7.0, 8.0 to 8.0
* Minimal supported MongoDB version is 6.0.17, 7.0.13, 8.0.0
* You can connect two Percona Server for MongoDB clusters, connect Percona Server for MongoDB (source) and Mongo Community/Enterprise Advanced cluster (target), connect Percona Server for MongoDB (source) and Atlas cluster (target)

## Data types

* Queryable encryption is not supported
* User and roles are not synchronized
* Timeseries collections are not supported
* InMemory Storage Engine is not supported
* Persistent Query Settings (for MongoDB 8)

## Other

* Multiple source or multiple target clusters
* Resumable initial synchronization
* Non-empty cluster support
* Database upgrade during the sync, even in the paused state.
* Reverse synchronization
* LDAP authentication
