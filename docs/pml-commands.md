# {{pml.full_name}} commands

## Overview

Percona MongoLink is a replication tool for MongoDB clusters. It provides commands to manage and monitor the replication process between source and target MongoDB clusters.

## Global options

These options are available for all commands:

| Option | Description | Default | Environment Variable |
|--------|-------------|---------|----------------------|
| `--log-level` | Set the logging level | `info` | - |
| `--log-json` | Output logs in JSON format | `false` | - |
| `--no-color` | Disable colored log output | `false` | - |
| `--port` | Server port number | `2242` | `PML_PORT` |

## Commands

### mongolink

The main command to start the MongoLink server.

**Required Options:**
- `--source`: MongoDB connection string for the source cluster (or `PML_SOURCE_URI` env var)
- `--target`: MongoDB connection string for the target cluster (or `PML_TARGET_URI` env var)

**Optional Options:**
- `--start`: Start Cluster Replication immediately (hidden)
- `--reset-state`: Reset stored MongoLink state (hidden)
- `--pause-on-initial-sync`: Pause on Initial Sync (hidden)

### version

Display the current version of Percona MongoLink.

```bash
mongolink version
```

### status

Get the status of the replication process.

```bash
mongolink status
```

### start

Start Cluster Replication.

```bash
mongolink start
```

**Optional Options:**
- `--pause-on-initial-sync`: Pause on Initial Sync (hidden)

### finalize

Finalize Cluster Replication.

```bash
mongolink finalize
```

**Optional Options:**
- `--ignore-history-lost`: Ignore history lost error (hidden)

### pause

Pause Cluster Replication.

```bash
mongolink pause
```

### resume

Resume Cluster Replication.

```bash
mongolink resume
```

### reset

Reset MongoLink state (hidden command).

**Subcommands:**
- `recovery`: Reset recovery state
- `heartbeat`: Reset heartbeat state

## Environment Variables

| Variable | Description | Default |
|----------|-------------|---------|
| `PML_SOURCE_URI` | MongoDB connection string for the source cluster | - |
| `PML_TARGET_URI` | MongoDB connection string for the target cluster | - |
| `PML_PORT` | Server port number | `2242` |
| `PML_USE_COLLECTION_BULK_WRITE` | Use Collection Bulk Write API | `false` |
| `PML_CLONE_NUM_PARALLEL_COLLECTIONS` | Number of collections cloned in parallel | `0` |
| `PML_CLONE_NUM_READ_WORKERS` | Number of read workers for cloning | `0` |
| `PML_CLONE_NUM_INSERT_WORKERS` | Number of insert workers for cloning | `0` |
| `PML_CLONE_SEGMENT_SIZE` | Segment size for cloning | Auto |
| `PML_CLONE_READ_BATCH_SIZE` | Read batch size for cloning | `0` |
| `PML_DEV_TARGET_CLIENT_COMPRESSORS` | Target client compressors (zstd,zlib,snappy) | - |

## Server Configuration

The server has the following default configuration:

- Default port: 2242
- Server read timeout: 30 seconds
- Server read header timeout: 3 seconds
- Maximum request size: 1 MiB
- Server response timeout: 5 seconds

## Replication States

The replication process can be in one of the following states:

- `failed`: The replication has failed
- `idle`: The replication is idle
- `running`: The replication is running
- `paused`: The replication is paused
- `finalizing`: The replication is finalizing
- `finalized`: The replication has been finalized