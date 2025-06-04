# {{pml.full_name}} commands

## Overview

Percona MongoLink is a replication tool for MongoDB clusters. It provides commands to manage and monitor the replication process between source and target MongoDB clusters.

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

Resets the `pml` state and deletes the metadata collections from target deployment. After the command execution, you must restart the `pml` service and start the data replication from scratch. Read more about the flow in [Troubleshooting guide](troubleshooting.md) 

```{.bash data-prompt="$"$}
$ pml reset --target
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

Available flags:

| Name | Description|
| -----| -----------|
| `--from-failure` | Resume replication from the last failure point |
