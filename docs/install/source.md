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

## Procedure

Here's how to build {{pml.full_name}}:
{.power-number}

1. Clone the repository:

    ```sh
    git clone https://github.com/percona-lab/percona-mongolink.git
    cd percona-mongolink
    ```

2. Build the project using the Makefile:

    ```sh
    make build
    ```

    Alternatively, you can install MongoLink from the cloned repo using `go install`:

    ```sh
    go install .
    ```

    > This will install `percona-mongolink` into your `GOBIN` directory. If `GOBIN` is included in your `PATH`, you can run MongoLink by typing `percona-mongolink` in your terminal.

3. Run the server:

    ```sh
    bin/percona-mongolink --source <source-mongodb-uri> --target <target-mongodb-uri>
    ```

    Alternatively, you can use environment variables:

    ```sh
    export SOURCE_URI=<source-mongodb-uri>
    export TARGET_URI=<target-mongodb-uri>
    bin/percona-mongolink --source $SOURCE_URI --target $TARGET_URI
    ```

    > Connections to the source and target must have `readPreference=primary` and `writeConcern=majority` explicitly or unset.

## Next steps

[Use {{pml.full_name}} :material-arrow-right:](usage.md){.md-button}