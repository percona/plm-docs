## Set up and start {{pml.full_name}}

# Configure authentication

Create users for PML in both source and target clusters:
{.power-number}

1. Connect to the source cluster and run the following command:

    ```javascript
    db.getSiblingDB('admin').createUser({
      user: 'source',
      pwd: 'mys3cretp@ss',
      roles: ['backup', 'clusterMonitor',  'readAnyDatabase'],
    });
    ```

2. Connect to the target cluster and run the following command:

    ```javascript
    db.getSiblingDB('admin').createUser({
       user: 'target',
       pwd: 'pass',
       roles: ['restore', 'clusterMonitor', 'clusterManager',       'readWriteAnyDatabase'],
      });
    ```


{{pml.full_name}} authenticates in source and target clusters using the MongoDB Connection string URI. It has the following format:

```
mongodb://user:pwd@host1:port1,host2:port2,host3:port3/[authdb]?readPreference=primary
```

If the password includes special characters like `#`, `@`, `/` and so on, you must convert these characters using the [percent-encoding mechanism](https://datatracker.ietf.org/doc/html/rfc3986#section-2.1) when passing them to Percona MongoLink. For example, the password `secret#pwd` should be passed as `secret%23pwd`.

!!! important

    Connections to the source and target must have `readPreference=primary` and `writeConcern=majority` either set explicitly or unset.

## Run Percona MongoLink

Run Percona MongoLink with the following command:

```{.bash data-prompt="$"}
bin/percona-mongolink --source <source-mongodb-uri>--target <target-mongodb-uri>
```

Alternatively, you can use environment variables:

```{.bash data-prompt="$"}
$ export SOURCE_URI=<source-mongodb-uri>
$ export TARGET_URI=<target-mongodb-uri>
$ bin/percona-mongolink --source $SOURCE_URI --target $TARGET_URI
```

## Next steps

[Use {{pml.full_name}} :material-arrow-right:](usage.md){.md-button}