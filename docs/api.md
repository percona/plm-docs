# How to use {{pml.full_name}} HTTP API?

## POST /start

Starts the replication process.

### Request 

- `includeNamespaces` (optional): List of namespaces to include in the replication.
- `excludeNamespaces` (optional): List of namespaces to exclude from the replication.

Example:

```json
curl -X POST http://localhost:2242/start -d '{
    "includeNamespaces": ["dbName.*", "anotherDB.collName1", "anotherDB.collName2"],
    "excludeNamespaces": ["dbName.collName"]
}'
```

### Response

- `ok`: Boolean indicating if the operation was successful.
- `error` (optional): Error message if the operation failed.

Example:

```json
{ "ok": true }
```

## POST /finalize

Finalizes the replication process.

### Request 

Example:

```json
curl -X POST http://localhost:2242/finalize
```

### Response

- `ok`: Boolean indicating if the operation was successful.
- `error` (optional): Error message if the operation failed.

Example:

```json
{ "ok": true }
```

## POST /pause

Pauses the replication process.

### Request 

Example:

```json
curl -X POST http://localhost:2242/pause
```

### Response

- `ok`: Boolean indicating if the operation was successful.
- `error` (optional): Error message if the operation failed.

Example:

```json
{ "ok": true }
```

## POST /resume

Resumes the replication process.

### Request

Example:

```json
curl -X POST http://localhost:2242/resume
```

### Response

- `ok`: Boolean indicating if the operation was successful.
- `error` (optional): Error message if the operation failed.

Example:

```json
{ "ok": true }
```

## GET /status

The /status endpoint provides the current state of the MongoLink replication process, including its progress, lag, and event processing details.

### Request

Example:

```json
curl -X GET http://localhost:2242/status
```

### Response

- `ok`: indicates if the operation was successful.
- `state`: the current state of the replication.
- `info`: provides additional information about the current state.
- `error` (optional): the error message if the operation failed.
- `lagTime`: the current lag time in logical seconds between source and target clusters.
- `eventsProcessed`: the number of events processed.
- `lastReplicatedOpTime`: the last replicated operation time.
- `initialSync.completed`: indicates if the initial sync is completed.
- `initialSync.lagTime`: the lag time in logical seconds until the initial sync completed.
- `initialSync.cloneCompleted`: indicates if the cloning process is completed.
- `initialSync.estimatedCloneSize`: the estimated total size of the clone.
- `initialSync.clonedSize`: the size of the data that has been cloned.

Example:

```json
{
    "ok": true,
    "state": "running",
    "info": "Initial Sync",

    "lagTime": 22,
    "eventsProcessed": 5000,
    "lastReplicatedOpTime": "1740335200.5",

    "initialSync": {
        "completed": false,
        "lagTime": 5,

        "cloneCompleted": false,
        "estimatedCloneSize": 5000000000,
        "clonedSize": 2500000000
    }
}
```
