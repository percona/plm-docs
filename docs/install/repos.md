# Install {{pml.full_name}} from Percona repositories

To install the software from Percona repositories means to subscribe to them. Percona provides the [`percona-release` :octicons-link-external-16:](https://www.percona.com/doc/percona-repo-config/index.html) repository management tool. It automatically enables the required repository so that you can install and update both {{pml.full_name}} packages and required dependencies smoothly.

Data migration is a resource-intensive task. Therefore, we recommend installing PML closest to the target to reduce the network lag as much as possible.

## Before you start

Check the [system requirements](../system-requirements.md) and [known limitations](../limitations.md).

## Procedure

<i warning>:material-alert: Warning:</i> Run the following commands as root or via the `sudo` command.

=== ":material-debian: On Debian and Ubuntu" 

    1. Install `percona-release`:

        --8<-- "percona-release-apt.md"

    2. Enable the repository    

        ```{.bash data-prompt="$"}
        $ sudo percona-release enable pml release
        ```

    3. Install the package:

        ```{.bash data-prompt="$"}
        $ sudo apt install percona-mongolink
        ```

=== ":material-redhat: On RHEL and derivatives" 

    1. Install `percona-release`:

        --8<-- "percona-release-yum.md"

    2. Enable the repository    

        ```{.bash data-prompt="$"}
        $ sudo percona-release enable pml release
        ```  

    3. Install the package:

        ```{.bash data-prompt="$"}
        $ sudo yum install percona-mongolink
        ``` 
    
Congratulations! You have successfully installed {{pml.full_name}}. Now you must connect it to source and target MongoDB deployments.

## Next steps

[Configure authentication :material-arrow-right:](authentication.md){.md-button}
