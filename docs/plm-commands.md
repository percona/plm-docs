# {{plm.full_name}} commands

## Overview

Percona Link for MongoDB is a replication tool for MongoDB clusters. It provides commands to manage and monitor the replication process between source and target MongoDB clusters.

## Commands

### version

Display the current version of Percona Link for MongoDB.

```{.bash data-prompt="$"$}
$ plm version
```

### status

Get the status of the replication process.

```{.bash data-prompt="$"$}
$ plm status
```

### start

Start cluster replication.

```{.bash data-prompt="$"$}
$ plm start
```

### reset

Resets the `PLM` state and deletes the metadata collections from target deployment. After the command execution, you must restart the `PLM` service and start the data replication from scratch. Read more about the flow in [Troubleshooting guide](troubleshooting.md) 

```{.bash data-prompt="$"$}
$ plm reset --target
```

### finalize

Finalize cluster replication.

```{.bash data-prompt="$"$}
$ plm finalize
```

### pause

Pause cluster replication.

```{.bash data-prompt="$"$}
$ plm pause
```

### resume

Resume cluster replication.

```{.bash data-prompt="$"$}
$ plm resume
```

Available flags:

| Name | Description|
| -----| -----------|
| `--from-failure` | Resume replication from the last failure point |
