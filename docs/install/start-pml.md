# Start Percona MongoLink

Start {{pml.full_name}}.

=== ":material-console: Using `systemd`"

    We recommend to use the packaged service scripts to run `pml`.
    
    ```{.bash data-prompt="$"}
    $ sudo systemctl start pml
    ```

    Check the status with this command:

    ```{.bash data-prompt="$"}
    $ sudo systemctl status pml
    ```

=== ":fontawesome-solid-user-gear: Manually"

    You can start PML manually. This option is the way you start {{pml.full_name}} if you installed it [from source code](source.md) 

    Run Percona MongoLink with the following command **if you haven't defined MongoDB connection string URI before**:

    ```{.bash data-prompt="$"}
    nohup pml --source <source-mongodb-uri> --target <target-mongodb-uri> --no-color > percona-mongolink.log 2>&1 &
    ```

    Alternatively, you can use environment variables:

    ```{.bash data-prompt="$"}
    $ export SOURCE_URI=<source-mongodb-uri>
    $ export TARGET_URI=<target-mongodb-uri>
    $ nohup pml --no-color > percona-mongolink.log 2>&1 &
 
    ```

## How to see {{pml.full_name}} logs

With the packaged `systemd` service, the log output to `stdout` is captured by
systemd’s default redirection to `systemd-journald`. You can view it with this
command:

```{.bash data-prompt="$"}
$ sudo journalctl -u pml.service
```

See `man journalctl` for useful options such as `--lines`, `--follow`, etc.


If you started `pbm-agent` manually, see the file you redirected `stdout` and `stderr` to.


## Next steps

Congratulations! you have successfully installed and connected PML to your source and target MongoDB. Now you have it up and running and you are ready to use it.

[Use {{pml.full_name}} :material-arrow-right:](usage.md){.md-button}