# Troubleshooting guide

## Recover {{pml.full_name}} from failure

When the `pml` service crashes and reports the `failed` state, you can recover it as follows:

1. Stop `pml` service:

    ```{.bash data-prompt="$"}
    $ sudo systemctl stop pml
    ```

2. Reset the pml state with the following command and pass the connection string URL to the target deployment:
 
    ```{.bash data-prompt="$"}
    $ pml reset --target <target-mongodb-uri>
    ```

    The command does the following:

    * Connects to the target MongoDB deployment
    * Deletes the metadata collections 
    * Restores the `pml` service from the `failed` state

3. Restart `pml`

    ```{.bash data-prompt="$"}
    $ sudo systemctl start pml
    ```

4. Start data replication from scratch:

    ```{.bash data-prompt="$"}
    $ pml start
    ```

