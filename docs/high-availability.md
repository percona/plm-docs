# High availability during replication

!!! admonition "Version added: 0.10.0"

Percona ClusterSync for MongoDB (PCSM) supports active-standby high availability during replication. Run two or more instances against the same source and target, and one of them takes charge while the rest wait. If the active instance becomes unavailable, another takes over and resumes replication from the last checkpoint.

High availability is always enabled and requires no configuration. A single instance behaves the same as in earlier versions. To enable failover, start a second instance with the same source and target.

!!! info "Important"
    High availability applies to the **replication phase after the initial clone completes**.

    The initial clone is not resumable. If the active PCSM instance fails during the clone, a standby becomes active, but the interrupted clone cannot continue. Start a new synchronization run to clone the data again.

## How high availability works

The instances coordinate through a lease stored on the target cluster, so the MongoDB deployment you already have is the only coordinator involved. Exactly one instance holds the lease at a time. That instance is `ACTIVE` and runs replication. The rest are `STANDBY` and do nothing until the lease expires.

PCSM uses three mechanisms to ensure safe failover:

### Lease election 

PCSM uses a lease to ensure that only one instance is ACTIVE at a time. Lease acquisition and renewal use atomic single-document operations. If several standby instances try to acquire an expired lease, only one can become active.

PCSM evaluates lease expiration using the target MongoDB server clock. Differences between the clocks on PCSM hosts therefore do not affect the election.

PCSM stores the lease as a single document in the `percona_clustersync_mongodb.lease` collection. For example:

    ```sh
    { "_id": "lease", "term": 7, "instanceId": "b3f1c2a4-9d7e-4c11-8a2f-1e6b0d5c9a77", "electionDate": { "$date": "2026-07-17T09:14:02.190Z" }, "expiresAt": { "$date": "2026-07-17T09:20:41.882Z" } }
    ```

Each instance also maintains a liveness document in the `percona_clustersync_mongodb.members` collection, refreshed on every heartbeat:

```{.json .no-copy}
{
  "_id": "b3f1c2a4-9d7e-4c11-8a2f-1e6b0d5c9a77",
  "group": "default",
  "host": "pcsm0",
  "port": 2242,
  "role": "ACTIVE",
  "term": 7,
  "pcsmVersion": "0.10.0",
  "startedAt": { "$date": "2026-07-17T09:14:02.113Z" },
  "lastHeartbeat": { "$date": "2026-07-17T09:20:31.882Z" }
}
```
A member whose `lastHeartbeat` falls past the stale threshold is treated as dead and drops out of the group view.

### Term fencing

Each lease has a term value that increases whenever a new `ACTIVE` instance is elected. PCSM includes this value in every checkpoint written by the active instance. 

If a previous active instance resumes after losing its lease, its checkpoint writes contain an outdated term and are rejected. The instance then switches to `STANDBY`, which prevents it from overwriting the current replication state.

### Checkpoint recovery

When a standby becomes `ACTIVE`, PCSM resumes replication from the last persisted checkpoint. Failover uses the existing recovery mechanism and happens automatically.

The timings are fixed:

| **Setting** | **Value** |
|---------|-------|
| Lease TTL | 10 seconds |
| Lease renewal by the active instance | Every 3 seconds |
| Heartbeat interval | Every 3 seconds |
| Stale member threshold | 3 missed heartbeats |

If the active instance stops unexpectedly, a standby can take over after the lease expires and continue replication from the latest checkpoint.

## Set up high availability

Run at least two PCSM instances on separate hosts, containers, or pods. Configure every instance with the same source and target clusters.

For example, run the following command on each host:

```bash
pcsm \
    --source "<source-mongodb-uri>" \
    --target "<target-mongodb-uri>"
```

No additional HA option is required.

If you run multiple PCSM instances on the same host, configure a different `--port` for each instance. For protection against a host failure, run the instances on separate hosts or pods.

See [Start PCSM](install/start-pcsm.md) for startup options and [Percona ClusterSync for MongoDB startup configuration](install/parameters.md) for the available parameters.

### Identify the HA group

Instances that share a group name coordinate as one active-standby group. Set the name with `--group-name` or the `PCSM_GROUP_NAME` environment variable:

```sh
pcsm \
    --source "<source-mongodb-uri>" \
    --target "<target-mongodb-uri>" \
    --group-name migration-1
```

The default group name is `default`. The name appears in member documents, in the API envelope, and as a label on the `..._ha_info` metric.

## Failover during the initial clone

High availability applies to the replication phase after the initial clone completes. The initial clone is not resumable because PCSM does not persist progress for individual collections.

If the `ACTIVE` instance becomes unavailable during the initial clone, a standby is promoted. During recovery, the new `ACTIVE` detects that the clone was interrupted and stops the synchronization. PCSM reports the reason in the logs and through the /status endpoint:

```sh
initial clone interrupted by failover and is not resumable; start a new run to re-clone from scratch
```

!!! info "Important"

    To recover, start a new synchronization run on the `ACTIVE` instance. PCSM starts the initial clone again from the beginning. Automatic recovery from the last checkpoint becomes available after the initial clone completes and PCSM enters the replication phase.

See PCSM HTTP API for information about the /status endpoint and Start and manage synchronization for information about starting a new synchronization run.

## Operate an HA deployment

In an HA deployment, you need to know which PCSM instance is ACTIVE, direct operational commands to that instance, and monitor the health of all instances. PCSM provides API responses, metrics, and health endpoints to help you manage these tasks.

### Check the active instance

PCSM exposes the HA role through the `/metrics` endpoint.

```bash
curl -sS http://localhost:2242/metrics | grep percona_clustersync_mongodb_ha_active
```

A value of `1` identifies the active instance. A value of `0` identifies a standby.

When PCSM sees more than one live member, API responses can also include the me, role, and group fields. These fields identify the instance that handled the request and list the other members of the HA group.

When PCSM sees more than one live member, API responses can also include the `me`, `role`, and `group` fields. These fields identify the instance that handled the request and list the other members of the HA group. For example, a `GET /status` response from the active instance:

```{.json .no-copy}
{
  "ok": false,
  "error": "not_active",
  "me": {
    "instanceId": "<instance-id>"
  },
  "role": "STANDBY",
  "group": {
    "term": 7,
    "members": [
      {
        "instanceId": "<instance-id>",
        "host": "pcsm0",
        "port": 2242,
        "role": "ACTIVE"
      },
      {
        "instanceId": "<instance-id>",
        "host": "pcsm1",
        "port": 2243,
        "role": "STANDBY"
      }
    ]
  }
}
```
A request sent to a standby returns HTTP `409` with the `not_active` error:

```{.json .no-copy}
{
  "ok": false,
  "error": "not_active",
  "me": { "instanceId": "6a2d8e10-4b3c-4f97-9c0a-2f7e1b4d6c88" },
  "role": "STANDBY",
  "group": {
    "name": "default",
    "term": 7,
    "members": [
      { "instanceId": "b3f1c2a4-9d7e-4c11-8a2f-1e6b0d5c9a77", "host": "pcsm0", "port": 2242, "role": "ACTIVE" },
      { "instanceId": "6a2d8e10-4b3c-4f97-9c0a-2f7e1b4d6c88", "host": "pcsm1", "port": 2243, "role": "STANDBY" }
    ]
  }
}
```

!!! info "Important"

    The `me`, `role`, and `group` fields are included only when the instance observes more than one live member. Applications that consume the PCSM API must therefore treat these fields as optional.

    A single PCSM instance continues to return API responses in the same format as earlier releases.

See the [PCSM HTTP API](api.md) for endpoint details.

###  Operational commands on standby instances

Replication commands must be sent to the active PCSM instance.

The following endpoints return HTTP `409` with `error: "not_active"` when called on a standby:

* `/status`
* `/start`
* `/pause`
* `/resume`
* `/finalize`

The response identifies the standby and, when available, includes the HA member list so you can locate the active instance.

The `/metrics` endpoint and `pprof` endpoints remain available on both active and standby instances.


See [PCSM commands](pcsm-commands.md) for information about managing a synchronization run.

## Configure readiness probes

Use `/metrics` for liveness and readiness probes in an HA deployment. This endpoint is available regardless of whether an instance is active or standby.

Do not use `/status` for a readiness probe. A healthy standby returns HTTP `409` from this endpoint because replication status is available only from the active instance.

!!! warning

    A healthy standby returns HTTP `409` from this endpoint because replication status is available only from the active instance. A probe pointed there marks every standby unhealthy. For Kubernetes deployments, see [Configure Liveness, Readiness and Startup Probes :octicons-link-external-16:](https://kubernetes.io/docs/tasks/configure-pod-container/configure-liveness-readiness-startup-probes/){:target="_blank"}.

For monitoring configuration, see [Set up observability with Percona Monitoring and Management](pmm-setup.md).

## High availability metrics

PCSM exposes the following HA metrics through `/metrics`:

| Metric                                                  | Description                                                                                                             |
| ------------------------------------------------------- | ----------------------------------------------------------------------------------------------------------------------- |
| `percona_clustersync_mongodb_ha_active`                 | Shows the current role. `1` means ACTIVE and `0` means STANDBY.                                                         |
| `percona_clustersync_mongodb_ha_term`                   | Shows the current HA lease term.                                                                                        |
| `percona_clustersync_mongodb_ha_role_transitions_total` | Counts role changes for the PCSM instance.                                                                              |
| `percona_clustersync_mongodb_ha_info`                   | Reports instance information. The metric has a constant value of `1` and includes the `instance_id` and `group` labels. |

You can use these metrics to identify the active instance, detect role changes, and monitor failover behavior.

## Reset HA state

PCSM provides commands to clear the stored HA membership or lease state.

!!! warning
    Stop all PCSM server instances that use the target cluster before running these commands. Do not reset HA state while PCSM is running.

Clear the recorded member information:

```bash
pcsm reset members
```

Clear the HA lease:

```bash
pcsm reset lease
```

Use these commands only when you need to clear HA coordination state. To clear all PCSM state, use `pcsm reset`.

## Upgrade from PCSM 0.9.0

Replication state created by PCSM 0.9.0 is not compatible with PCSM 0.10.0.

Before starting PCSM 0.10.0:
{.power-number}

1. Stop all PCSM 0.9.0 instances that use the target cluster.

2.Rreset the stored PCSM state on the target:

    ```bash
    pcsm reset --target "<target-mongodb-uri>"
    ```

3. Start PCSM 0.10.0
4. Start a new synchronization run.

!!! important
    Do not run PCSM 0.9.0 and PCSM 0.10.0 against the same target at the same time.

## Next steps

[Manage synchronization with PCSM commands](pcsm-commands.md){.md-button}

[Use the PCSM HTTP API](api.md){.md-button}

[Set up observability with Percona Monitoring and Management](pmm-setup.md){.md-button}


