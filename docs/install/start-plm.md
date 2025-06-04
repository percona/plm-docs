# Start Percona Link for MongoDB

Start {{PLM.full_name}}.

=== ":material-console: Using `systemd`"

    We recommend to use the packaged service scripts to run `PLM`.
    
    ```{.bash data-prompt="$"}
    $ sudo systemctl start PLM
    ```

    Check the status with this command:

    ```{.bash data-prompt="$"}
    $ sudo systemctl status PLM
    ```

=== ":fontawesome-solid-user-gear: Manually"

    You can start PLM manually. This option is the way you start {{PLM.full_name}} if you installed it [from source code](source.md) 

    Run Percona Link for MongoDB with the following command **if you haven't defined MongoDB connection string URI before**:

    ```{.bash data-prompt="$"}
    $ nohup PLM --source <source-mongodb-uri> --target <target-mongodb-uri> --no-color > percona-link-mongodb.log 2>&1 &
    ```

    Alternatively, you can use environment variables:

    ```{.bash data-prompt="$"}
    $ export SOURCE_URI=<source-mongodb-uri>
    $ export TARGET_URI=<target-mongodb-uri>
    $ nohup PLM --no-color > percona-link-mongodb.log 2>&1 &
    ```

See [Percona Link for MongoDB startup configuration](parameters.md) for all available options.


## How to see {{PLM.full_name}} logs

With the packaged `systemd` service, the log output to `stdout` is captured by
systemd’s default redirection to `systemd-journald`. You can view it with this
command:

```{.bash data-prompt="$"}
$ sudo journalctl -u PLM.service
```

See `man journalctl` for useful options such as `--lines`, `--follow`, etc.


If you started `PLM` manually, see the file you redirected `stdout` and `stderr` to.


## Next steps

Congratulations! you have successfully installed and connected PLM to your source and target MongoDB. Now you have it up and running and you are ready to use it.

[Use {{PLM.full_name}} :material-arrow-right:](usage.md){.md-button}