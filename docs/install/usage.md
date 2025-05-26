# Use {{pml.full_name}}

{{pml.full_name}} doesn't automatically start data replication after the startup. It has the `idle` status indicating that it is ready to accept requests.

You can interact with {{pml.full_name}} using the command-line interface or via the HTTP API. Read more about [PML API](../api.md).

## Before you start

Your target MongoDB cluster may be empty or contain data. PML replicates data from the source to the target but doesn’t manage the target’s data. If the target already has the same data as the source, PML overwrites it. However, if the target contains different data, PML doesn't delete it during replication. This leads to inconsistencies between the source and target. To ensure consistency, manually delete any existing data from the target before starting replication.

## Start the replication

Start the replication process between source and target clusters. PML starts copying the data from the source to the target. First it does the initial sync by cloning the data and then applying all the changes that happened since the clone start. 

Then it uses the [change streams :octicons-link-external-16:](https://www.mongodb.com/docs/manual/changeStreams/) to track the changes to your data on the source and replicate them to the target.

=== "Command line"

    ```{.bash data-prompt="$"}
    $ pml start
    ```

    ??? example "Expected output"

        ```{.json .no-copy}
        {
          "ok": true
        }
        ```

=== "HTTP API"
    
    Send a POST request to the `/start` endpoint:

    ```{.bash data-prompt="$"}
    $ curl -X POST http://localhost:2242/start 
    ```

## Start the filtered replication

You can replicate the whole dataset or specific namespaces - databases and collections. You can specify what namespaces to include and/or exclude from the replication. Currently you can start the filtered replication only via the API. The ability to start it via the CLI will be added in future releases.

=== "HTTP API"
    
    Send a POST request to the `/start` endpoint:

    ```{.bash data-prompt="$"}
    $ curl -X POST http://localhost:2242/start -d '{
        "includeNamespaces": ["db1.collection1", "db2.collection2"],
        "excludeNamespaces": ["db3.collection3"]
    }'
    ```

## Pause the replication

You can pause the replication at any moment. PML stops the replication, saves the timestamp and enters the `paused` state. PML uses the saved timestamp after you [resume the replication](#resume-the-replication).

=== "Command line"

    ```{.bash data-prompt="$"}
    $ pml pause
    ```

=== "HTTP API"

    Send a POST request to the `/pause` endpoint:

    ```{.bash data-prompt="$"}
    $ curl -X POST http://localhost:2242/pause
    ```

## Resume the replication

Resume the replication. PML changes the state to `running` and copies the changes that occurred to the data from the timestamp it saved when you paused the replication. Then it continues monitoring the data changes and replicating them real-time. 

=== "Command line"

    ```{.bash data-prompt="$"}
    $ pml resume
    ```

=== "HTTP API"

    Send a POST request to the `/resume` endpoint:

    ```{.bash data-prompt="$"}
    $ curl -X POST http://localhost:2242/resume
    ```

The replication may fail for some reason, like lost connectivity or the like. In this case you can resume replication by adding the `--from-failure` flag to the `resume` command:

=== "Command line"

    ```{.bash data-prompt="$"}
    $ pml resume
    ```

=== "HTTP API"

    Send a POST request to the `/resume` endpoint:

    ```{.bash data-prompt="$"}
    $ curl -X POST http://localhost:2242/resume -d '{
        "fromFailure": true
    }'
    ```


## Check the replication status

Check the current status of the replication process.

=== "Command line"

    ```{.bash data-prompt="$"}
    $ pml status
    ```

=== "HTTP API"

    Send a GET request to the `/status` endpoint:

    ```{.bash data-prompt="$"}
    $ curl http://localhost:2242/status
    ```

# Finalize the replication

When you no longer need / want to replicate data, finalize the replication. PML stops replication, creates the required indexes on the target, and stops. This is a one-time operation. You cannot restart the replicaton after you finalized it. If you run the `start` command, PML will start the replication anew, with the initial sync. 

=== "Command line"

    ```{.bash data-prompt="$"}
    $ pml finalize
    ```

=== "HTTP API"
    
    Send a POST request to the `/finalize` endpoint:

    ```{.bash data-prompt="$"}
    $ curl -X POST http://localhost:2242/finalize
    ```