# Replicate from one source to multiple targets

!!! admonition "Version added: 0.10.0"

You can run multiple {{pcsm.full_name}} (PCSM) instances against the same source cluster and replicate different namespaces to different target clusters at the same time. This lets you split one cluster across several destinations in a single pass, with each instance moving only the data you assign to it.

## How it works

Each PCSM instance has:

- The same source cluster
- A different target cluster
- Its own namespace filter

Every instance runs the full replication workflow on its own: clone, replication, and finalization. See [How PCSM works](intro.md) for what happens at each stage.

!!! note

    You run `start`, `status`, and `finalize` against each instance separately. Instances in separate containers can all use the default port 2242. See [Percona ClusterSync for MongoDB startup configuration](install/parameters.md) for the available options.

The examples below replicate `db_0` to the first target and `db_1` to the second. Select the tab that matches your deployment.

!!! warning "Technical preview"

    Sharding support in PCSM is a technical preview and is not recommended for production. See [Sharding support in Percona ClusterSync for MongoDB](sharding.md).

=== "Replica set"

    ## Replicate from a replica set to two targets

    This example uses a source replica set `rs1` and two target replica sets, `rs2` and `rs3`.

    | **PCSM instance** | **Source** | **Target** | **Included namespaces** |
    |-------------------|------------|------------|-------------------------|
    | csync-a           | rs1        | rs2        | `db_0.*`                |
    | csync-b           | rs1        | rs3        | `db_1.*`                |

    Follow these steps:
    {.power-number}

    1. Start `csync-a` with `rs1` as the source and `rs2` as the target:

        ```bash
                pcsm \
                --source "mongodb://csync:<password>@rs101:27017/?replicaSet=rs1" \
                --target "mongodb://csync:<password>@rs201:27017/?replicaSet=rs2"
        ```

    2. Start `csync-b` against the same source, with `rs3` as the target:

        ```bash
                pcsm \
                --source "mongodb://csync:<password>@rs101:27017/?replicaSet=rs1" \
                --target "mongodb://csync:<password>@rs301:27017/?replicaSet=rs3"
        ```

    3. Start replication on `csync-a`, filtered to the namespaces it replicates:

        ```bash
                pcsm start --include-namespaces="db_0.*"
        ```

        ??? example "Expected output"

            ```{.json .no-copy}
                        {
                        "ok": true
                        }
            ```

    4. Start replication on `csync-b` with its own filter:

        ```bash
                pcsm start --include-namespaces="db_1.*"
        ```

        For how include and exclude filters interact, see [Start the filtered replication](install/usage.md#start-the-filtered-replication). For the full flag list, see [PCSM commands](pcsm-commands.md). You can also drive every step through the [PCSM HTTP API](api.md).

    5. Check each instance and wait for the clone and replication stages to complete:

        ```bash
                pcsm status
        ```

    6. Finalize each instance:

        ```bash
                pcsm finalize
        ```

    7. Check the status of each instance after finalization. The following output is from `csync-a`. The `csync-b` output has the same structure with its own operation time and finalization timestamps:

        ```bash
                pcsm status
        ```

        ??? example "Expected output"

            ```{.json .no-copy}
                        {
                        "ok": true,
                        "state": "finalized",
                        "info": "Finalized",
                        "lagTimeSeconds": 0,
                        "eventsRead": 0,
                        "eventsApplied": 0,
                        "lastReplicatedOpTime": {
                            "ts": "1787298593.1",
                            "isoDate": "2026-08-21T07:49:53Z"
                        },
                        "initialSync": {
                            "estimatedCloneSizeBytes": 9877780,
                            "clonedSizeBytes": 9877780,
                            "completed": true,
                            "cloneCompleted": true
                        },
                        "finalization": {
                            "completed": true,
                            "startedAt": "2026-08-21T07:49:53.633159569Z",
                            "completedAt": "2026-08-21T07:49:53.759444616Z"
                        }
                        }
            ```

    ### Verify the result on replica set targets

    Connect to each target and confirm it holds only the namespaces that its instance replicated.

    **On `rs2`.** List the databases, then count the documents. The `db_0` database returns the full count and `db_1` returns zero:

        ```javascript
            show databases
            db.getSiblingDB('db_0').docs.countDocuments({})
            db.getSiblingDB('db_1').docs.countDocuments({})
        ```

    Check the indexes that PCSM recreated on the target:

        ```javascript
            db.getSiblingDB('db_0').docs.getIndexes().map(i => i.name)
        ```

    ??? example "Expected output"

        ```{.text .no-copy}
                [
                '_id_',
                'value_1',
                'value_1_uid_-1',
                'uid_1',
                'tag_text',
                '_id_hashed',
                'created_at_1',
                'value_partial',
                'tag_sparse'
                ]
        ```

    The collection replicated to the other target does not exist here, so querying it returns an error. This is the expected result:

    ```javascript
        db.getSiblingDB('db_1').docs.getIndexes().map(i => i.name)
    ```

    ```{.text .no-copy}
        MongoServerError[NamespaceNotFound]: ns does not exist: db_1.docs
    ```

    **On `rs3`.** Run the same checks with the databases reversed. Here `db_1` holds the data, and querying `db_0.docs` returns `ns does not exist: db_0.docs`.

    Finally, check the logs of each instance for errors. See [Logging in Percona ClusterSync for MongoDB](logging.md).

=== "Sharded cluster"

    ## Replicate from a sharded cluster to two targets

    This example uses three sharded clusters, each with its own [mongos :octicons-link-external-16:](https://www.mongodb.com/docs/manual/core/sharded-cluster-query-router/){:target="_blank"}, config server, and two shards. One cluster is the source and two are targets.

    | **PCSM instance** | **Source** | **Target** | **Included namespaces** |
    |-------------------|------------|------------|-------------------------|
    | csync-a           | mongos1    | mongos2    | `db_0.*`                |
    | csync-b           | mongos1    | mongos3    | `db_1.*`                |

    PCSM connects through `mongos` on both the source and the target, so you do not need to list individual shard members or config servers in the connection strings.
    {.power-number}

    1. Start `csync-a` against the source `mongos` and the first target `mongos`:

        ```bash
                pcsm \
                --source "mongodb://csync:<password>@mongos1:27017" \
                --target "mongodb://csync:<password>@mongos2:27017"
        ```

    2. Start `csync-b` against the same source `mongos` and the second target `mongos`:

        ```bash
                pcsm \
                --source "mongodb://csync:<password>@mongos1:27017" \
                --target "mongodb://csync:<password>@mongos3:27017"
        ```

    3. Start replication on `csync-a`:

        ```bash
                pcsm start --include-namespaces="db_0.*"
        ```

        ??? example "Expected output"

            ```{.json .no-copy}
                        {
                        "ok": true
                        }
            ```

    4. Start replication on `csync-b`:

        ```bash
        pcsm start --include-namespaces="db_1.*"
        ```

    5. Check each instance and wait for the clone and replication stages to complete:

        ```bash
        pcsm status
        ```

    6. Finalize each instance:

    ```bash
    pcsm finalize
    ```

    7. Check the status of each instance after finalization. The following output is from `csync-a`:

    ```bash
    pcsm status
    ```

        ??? example "Expected output"

            ```{.json .no-copy}
                        {
                        "ok": true,
                        "state": "finalized",
                        "info": "Finalized",
                        "lagTimeSeconds": 2,
                        "eventsRead": 6,
                        "eventsApplied": 5,
                        "lastReplicatedOpTime": {
                            "ts": "1787301347.3",
                            "isoDate": "2026-08-21T08:35:47Z"
                        },
                        "initialSync": {
                            "estimatedCloneSizeBytes": 9877780,
                            "clonedSizeBytes": 9877780,
                            "completed": true,
                            "cloneCompleted": true
                        },
                        "finalization": {
                            "completed": true,
                            "startedAt": "2026-08-21T08:35:47.949454942Z",
                            "completedAt": "2026-08-21T08:35:48.21823288Z"
                        }
                        }
            ```

    ### Verify the result on sharded targets

    Connect to the `mongos` of each target cluster and confirm it holds only the namespaces that its instance replicated.

    **On `mongos2`.** List the databases:

        ```javascript
            show databases
        ```

    ??? example "Expected output"

        ```{.text .no-copy}
                admin                        172.00 KiB
                config                         2.11 MiB
                db_0                          31.56 MiB
                percona_clustersync_mongodb  168.00 KiB
        ```

    Count the documents. The `db_0` database returns the full count and `db_1` returns zero:

        ```javascript
            db.getSiblingDB('db_0').docs.countDocuments({})
            db.getSiblingDB('db_1').docs.countDocuments({})
        ```

    Check the indexes:

        ```javascript
            db.getSiblingDB('db_0').docs.getIndexes().map(i => i.name)
        ```

    ??? example "Expected output"

        ```{.text .no-copy}
                [
                '_id_',
                'value_1',
                    'value_1_uid_-1',
                    'uid_1',
                    'tag_text',
                    '_id_hashed',
                    'created_at_1',
                    'value_partial',
                    'tag_sparse'
                    ]
            ```

    The collection replicated to the other target does not exist here, so querying it returns an error. This is the expected result:

        ```javascript
            db.getSiblingDB('db_1').docs.getIndexes().map(i => i.name)
        ```

        ```{.text .no-copy}
            MongoServerError[NamespaceNotFound]: ns does not exist: db_1.docs
        ```

    **On `mongos3`.** List the databases:

        ```javascript
            show databases
        ```

    ??? example "Expected output"

        ```{.text .no-copy}
                admin                        172.00 KiB
                config                         2.11 MiB
                db_1                          31.55 MiB
                percona_clustersync_mongodb  168.00 KiB
        ```

    Count the documents. The `db_1` database returns the full count and `db_0` returns zero:

        ```javascript
            db.getSiblingDB('db_1').docs.countDocuments({})
            db.getSiblingDB('db_0').docs.countDocuments({})
        ```

    Check the indexes:

        ```javascript
            db.getSiblingDB('db_1').docs.getIndexes().map(i => i.name)
        ```

    Querying the collection replicated to the other target returns an error:

        ```javascript
            db.getSiblingDB('db_0').docs.getIndexes().map(i => i.name)
        ```

        ```{.text .no-copy}
            MongoServerError[NamespaceNotFound]: ns does not exist: db_0.docs
        ```

    Finally, check the logs of each instance for errors. See [Logging in Percona ClusterSync for MongoDB](logging.md).