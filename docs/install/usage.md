# Use {{pml.full_name}}

You can  {{pml.full_name}} either via the command-line interface or via the HTTP REST API. Read more about [PML API](../api.md).

## Start the Replication

=== "Command line"

    ```{.bash data-prompt="$"}
    $ bin/percona-mongolink start
    ```

=== "HTTP API"
    
    Send a POST request to the `/start` endpoint with the desired options:

    ```{.bash data-prompt="$"}
    $ curl -X POST http://localhost:2242/start -d '{
        "includeNamespaces": ["db1.collection1", "db2.collection2"],
        "excludeNamespaces": ["db3.collection3"]
    }'
    ```

## Pause the replication

=== "Command line"

    ```{.bash data-prompt="$"}
    $ bin/percona-mongolink pause
    ```

=== "HTTP API"

    Send a POST request to the `/pause` endpoint:

    ```{.bash data-prompt="$"}
    $ curl -X POST http://localhost:2242/pause
    ```

## Resume the replication

=== "Command line"

    ```{.bash data-prompt="$"}
    $ bin/percona-mongolink resume
    ```

=== "HTTP API"

    Send a POST request to the `/resume` endpoint:

    ```{.bash data-prompt="$"}
    $ curl -X POST http://localhost:2242/resume
    ```

## Check the replication status

Check the current status of the replication process.

=== "Command line"

    ```{.bash data-prompt="$"}
    $ bin/percona-mongolink status
    ```

=== "HTTP API"

    Send a GET request to the `/status` endpoint:

    ```{.bash data-prompt="$"}
    $ curl http://localhost:2242/status
    ```

## Finalize the replication

=== "Command line"

    ```{.bash data-prompt="$"}
    $ bin/percona-mongolink finalize
    ```

=== "HTTP API"
    
    Send a POST request to the `/finalize` endpoint:

    ```{.bash data-prompt="$"}
    $ curl -X POST http://localhost:2242/finalize
    ```
