# Configure authentication in MongoDB

{{pml.full_name}} uses the authentication and authorization subsystem of MongoDB. This means that to authenticate Percona Backup for MongoDB, you need to:

* [Create users in source and target cluster](#create-users)
* [Set a valid MongoDB connection string URI for source and target cluster](#set-mongodb-connection-string-uri)

## Create users

You need to create users in both source and target clusters. You will use these user credentials to create the MongoDB Connection string URI.
{.power-number}

1. Connect to the **source** cluster and run the following command:

    ```javascript
    db.getSiblingDB('admin').createUser({
      user: 'source',
      pwd: 'mys3cretpAss',
      roles: ['backup', 'clusterMonitor',  'readAnyDatabase'],
    });
    ```

2. Connect to the **target** cluster and run the following command:

    ```javascript
    db.getSiblingDB('admin').createUser({
       user: 'target',
       pwd: 'pass',
       roles: ['restore', 'clusterMonitor', 'clusterManager',       'readWriteAnyDatabase'],
      });
    ```

## Set MongoDB connection string URI

{{pml.full_name}} authenticates in source and target clusters using the MongoDB Connection string URI. It has the following format:

```
mongodb://user:pwd@host1:port1,host2:port2,host3:port3/[authdb]?readPreference=primary
```

To connect PML to source and target MongoDB clusters, specify the MongoDB Connection string URI for the `PML_SOURCE_URI` and `PML_TARGET_URI` environment variables in its environment file. 

When you [install PML from repositories](repos.md), the environment file is created for you. You can find it at the following path:

* for Debian and Ubuntu: `/etc/default/percona-mongolink`
* for RHEL and derivatives: `/etc/sysconfig/percona-mongolink`

### Example environment file 

```{.text .no-copy}
PML_SOURCE_URI="source:mys3cretpAssword@mysource1:27017,mysource2:27017,mysource3:27017/admin?readPreference=primary"
PML_TARGET_URI="target:pass@mytarget1:27017,mytarget2:27017,mytarget3:27017/admin?readPreference=primary"
```

### Passwords with special characters

If the password includes special characters like `#`, `@`, `/` and so on, you must convert these characters using the [percent-encoding mechanism](https://datatracker.ietf.org/doc/html/rfc3986#section-2.1) when passing them to Percona MongoLink. For example, the password `secret#pwd` should be passed as `secret%23pwd`.

## Next steps 

[Start PML :material-arrow-right: ](start-pml.md){.md-button}