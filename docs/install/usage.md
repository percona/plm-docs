# How to use {{pml.full_name}}?

## Starting the Replication

To start the replication process, you can either use the command-line interface or send a POST request to the `/start` endpoint with the desired options:

### Using Command-Line Interface

```sh
bin/percona-mongolink start
```

### Using HTTP API

```sh
curl -X POST http://localhost:2242/start -d '{
    "includeNamespaces": ["db1.collection1", "db2.collection2"],
    "excludeNamespaces": ["db3.collection3"]
}'
```

## Finalizing the Replication

To finalize the replication process, you can either use the command-line interface or send a POST request to the `/finalize` endpoint:

### Using Command-Line Interface

```sh
bin/percona-mongolink finalize
```

### Using HTTP API

```sh
curl -X POST http://localhost:2242/finalize
```

## Pausing the Replication

To pause the replication process, you can either use the command-line interface or send a POST request to the `/pause` endpoint:

### Using Command-Line Interface

```sh
bin/percona-mongolink pause
```

### Using HTTP API

```sh
curl -X POST http://localhost:2242/pause
```

## Resuming the Replication

To resume the replication process, you can either use the command-line interface or send a POST request to the `/resume` endpoint:

### Using Command-Line Interface

```sh
bin/percona-mongolink resume
```

### Using HTTP API

```sh
curl -X POST http://localhost:2242/resume
```

## Checking the Status

To check the current status of the replication process, you can either use the command-line interface or send a GET request to the `/status` endpoint:

### Using Command-Line Interface

```sh
bin/percona-mongolink status
```

### Using HTTP API

```sh
curl http://localhost:2242/status
```