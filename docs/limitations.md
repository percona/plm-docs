---
title: Known issues and limitations
author: Radoslaw Szulgo
---
# Known issues and limitations

This page lists known limitations for using Percona MongoLink

## Versions and topology

* Sharded clusters are not supported
* MongoDB versions that reached End-of-Life are not supported
* PML connects only to the primary node in the replica set. You cannot force connection to secondary members using the [directConnection :octicons-link-external-16:](https://www.mongodb.com/docs/manual/reference/connection-string/#connection-string-formats) option. This option is ignored.



## Data types

* Queryable encryption is not supported
* Users and roles are not synchronized
* Timeseries collections are not supported
* [Percona Memory Engine :octicons-link-external-16:](https://docs.percona.com/percona-server-for-mongodb/8.0/inmemory.html) is not supported
* Persistent Query Settings (added in MongoDB 8) are not supported 
* documents that have [field names with periods and dollar signs :octicons-link-external-16:](https://www.mongodb.com/docs/manual/core/dot-dollar-considerations/) are not supported

## Other

The following functionalities are not supported:

* Multiple source or multiple target clusters 
* You cannot resume initial synchronization if an issue occurred. You must start it from scratch.
* Database upgrade during the sync, even in the paused state.
* Reverse synchronization
* External authentication via Kerberos, AWS and LDAP
* [GridFS :octicons-link-external-16:](https://www.mongodb.com/docs/manual/core/gridfs/)
