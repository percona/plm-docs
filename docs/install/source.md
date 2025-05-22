# Build from source code

## Before you start

Check the [system requirements](../system-requirements.md)  and [known limitations](../limitations.md).

## Prerequisites

To build {{pml.full_name}} from source, you need the following:

- Go 1.24 or later. [Install and set up Go tools :octicons-link-external-16:](https://golang.org/doc/install)x
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

3. Add `GOBIN` to your `PATH`. This makes `percona-mongolink` a global command in your terminal.

## Next steps

[Configure authentication :material-arrow-right:](authentication.md){.md-button}