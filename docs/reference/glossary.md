# Glossary

## ACID
     
Set of properties that guarantee database transactions are processed reliably. Stands for [`Atomicity`](#atomicity), [`Consistency`](#consistency), [`Isolation`](#isolation), [`Durability`](#durability).

## Atomicity

Atomicity means that database operations are applied following an "all or nothing" rule. A transaction is either fully applied or not at all.

## Blob
    
A blob stands for Binary Large Object, which includes objects such as images and multimedia files. In other words these are various data files that you store in Microsoft's data storage platform. Blobs are organized in [containers](#container) which are kept in Azure Blob storage under your storage account.

## Bucket

A bucket is a container on the s3 remote storage that stores backups.

## Collection
     
A collection is the way data is organized in MongoDB. It is analogous to a table in relational databases.

## Consistency

In the context of backup and restore, consistency means that the data restored will be consistent in a given point in time. Partial or incomplete writes to disk of atomic operations (for example, to table and index data structures separately) won't be served to the client after the restore. The same applies to multi-document transactions that started but didn't complete by the time the backup was finished.

## Container

A container in Microsoft Azure Blob storage organizes a set of [blobs](#blob), similar to a directory in a file system. It can include an unlimited number of blobs.

A container name must be a valid DNS name, as it forms part of the unique URI (Uniform resource identifier) used to address the container or its blobs. 

## Durability
   
Once a transaction is committed, it will remain so.

## Isolation

The Isolation requirement means that no transaction can interfere with another.

## Percona Server for MongoDB 

Percona Server for MongoDB is a drop-in replacement for MongoDB Community Edition with enterprise-grade features.

## Replica set
   
A replica set is a group of `mongod` nodes that host the same data set.

## Technical preview feature

Technical preview features are not yet ready for enterprise use and are not included in support via SLA. They are included in this release so that users can provide feedback prior to the full release of the feature in a future GA release (or removal of the feature if it is deemed not useful). This functionality can change (APIs, CLIs, etc.) from tech preview to GA. 

