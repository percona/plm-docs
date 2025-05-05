# Build from source code

## Before you start

Check the [system requirements](../system-requirements.md)

## Prerequisites

To build {{pml.full_name}} from source, you need the following:

- Go 1.24 or later. [Install and set up Go tools :octicons-link-external-16:](https://golang.org/doc/install)
- MongoDB 6.0 or later
- Python 3.13 or later (for testing)
- Poetry (for managing Python dependencies)
- make
- git

## Build steps

Here's how to build {{pml.full_name}}:
{.power-number}

1. Clone the repository and change directory to `percona-mongolink`:

    ```{.bash data-prompt="$"}
    $ git clone https://github.com/percona-lab/percona-mongolink.git
    $ cd percona-mongolink
    ```

2. Build the project using the Makefile:

    ```{.bash data-prompt="$"}
    $ make build
    ```

    Alternatively, you can install MongoLink from the cloned repo using `go install`:

    ```{.bash data-prompt="$"}
    $ go install .
    ```

    This installs `percona-mongolink` into your `GOBIN` directory. 

3. Add `GOBIN` to your `PATH` and you can run {{pml.full_name}} by typing `percona-mongolink` in your terminal.

## Configure authentication

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