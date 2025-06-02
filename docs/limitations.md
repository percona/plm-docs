---
title: Known issues and limitations
author: Radoslaw Szulgo
---
# Known issues and limitations

This page lists known limitations for using Percona MongoLink

## Versions and topology

* Sharded clusters are not supported
* MongoDB versions that reached End-of-Life are not supported
* Percona MongoLink supports only Replica Set to Replica Set synchronization. The source and target replica sets can have different number of nodes
* You can synchronize Percona Server for MongoDB or MongoDB Community/Enterprise Advanced/Atlas within the same major versions - 6.0 to 6.0, 7.0 to 7.0, 8.0 to 8.0
* Minimal supported MongoDB versions are: 6.0.17, 7.0.13, 8.0.0
* You can connect the following MongoDB deployments:

   | Source | Target |
   | --- | --- |
   | Percona Server for MongoDB | Percona Server for MongoDB |
   | Percona Server for MongoDB | MongoDB Community |
   | Percona Server for MongoDB | MongoDB Enterprise Advanced |
   | Percona Server for MongoDB | MongoDB Atlas |



## Data types

* Queryable encryption is not supported
* Users and roles are not synchronized
* Timeseries collections are not supported
* [Percona Memory Engine](https://docs.percona.com/percona-server-for-mongodb/8.0/inmemory.html) is not supported
* Persistent Query Settings (added in MongoDB 8) are not supported 

## Other

The following functionalities are not supported:

* Multiple source or multiple target clusters 
* Resumable initial synchronization 
* Synchronization with a non-empty target cluster
* Database upgrade during the sync, even in the paused state.
* Reverse synchronization
* LDAP authentication
