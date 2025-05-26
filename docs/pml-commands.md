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

### version

Display the current version of Percona MongoLink.

```{.bash data-prompt="$"$}
$ pml version
```

### status

Get the status of the replication process.

```{.bash data-prompt="$"$}
$ pml status
```

### start

Start cluster replication.

```{.bash data-prompt="$"$}
$ pml start
```

### reset

Resets the `pml` state and deletes the metadata collections from target deployment. To use this command you must first stop the `pml` service. Then, you must restart `pml` and start the data replication from scratch. Read more about the flow in [Troubleshooting guide](troubleshooting.md) 

```{.bash data-prompt="$"$}
pml reset --target
```



### finalize

Finalize cluster replication.

```{.bash data-prompt="$"$}
$ pml finalize
```

### pause

Pause cluster replication.

```{.bash data-prompt="$"$}
$ pml pause
```

### resume

Resume cluster replication.

```{.bash data-prompt="$"$}
$ pml resume
```

