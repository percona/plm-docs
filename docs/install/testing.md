# How to test {{pml.full_name}}?

## Prerequisites

- Install Poetry:

    ```sh
    curl -sSL https://install.python-poetry.org | python3 -
    ```

- Install the required Python packages:

    ```sh
    poetry install
    ```

### Build for Testing

To build the project for testing, use the following command:

```sh
make test-build
```

### Running Tests

To run the tests, use the following command:

```sh
poetry run pytest \
    --source-uri <source-mongodb-uri> \
    --target-uri <target-mongodb-uri> \
    --mongolink-url http://localhost:2242 \
    --mongolink-bin bin/percona-mongolink_test
```

Alternatively, you can use environment variables:

```sh
export TEST_SOURCE_URI=<source-mongodb-uri>
export TEST_TARGET_URI=<target-mongodb-uri>
export TEST_MONGOLINK_URL=http://localhost:2242
export TEST_MONGOLINK_BIN=bin/percona-mongolink_test
poetry run pytest
```

> The `--mongolink-bin` flag or `TEST_MONGOLINK_BIN` environment variable specifies the path to the MongoLink binary. This allows the test suite to manage the MongoLink process, ensuring it starts and stops as needed during the tests. If neither the flag nor the environment variable is provided, you must run MongoLink externally before running the tests.