# Runtime Commands (`start`, `stop`, `restart`, `status`, `logs`)

These commands manage ScipionAPI runtime services.

They control:

- **FastAPI / uvicorn**
- **Celery worker**

Services are typically started as detached background processes when using the CLI runtime commands.

!!! note "Production recommendation"
    For production deployments, prefer `systemd` instead of CLI-managed background processes.

---

## Command Summary

| Command | Purpose |
|---|---|
| `start` | Start API and Celery |
| `stop` | Stop API and Celery |
| `restart` | Restart both services |
| `status` | Show whether services are running |
| `logs` | Tail runtime logs |

---

## Recommended mental model

A practical sequence is:

1. `start` to launch services
2. `status` to confirm process state
3. `logs` to verify real runtime behavior
4. `restart` after configuration changes
5. `stop` when shutting down intentionally

`status` tells you whether processes exist. `logs` and `/health` tell you whether the system is actually behaving well.

---

## `start`

Starts the API and Celery worker.

```
./scripts/scipionapi start
```

Typical behavior:

- launches API and Celery as detached processes
- creates PID files under `.run/`

---

## `stop`

Stops API and Celery.

```
./scripts/scipionapi stop
```

Typical behavior:

- reads PID files
- terminates the running processes when possible

---

## `restart`

Stops and starts services again.

```
./scripts/scipionapi restart
```

Useful after:

- `.env` changes
- dependency or runtime updates
- integrated Web asset redeployments
- transient failures

---

## `status`

Shows whether services are running.

```
./scripts/scipionapi status
```

!!! note "Process state vs health"
    `status` confirms process state, but not necessarily application health. Also check logs and `/health`.

---

## `logs`

Tails application logs.

```
./scripts/scipionapi logs
```

This typically shows the API log and the worker log together.

---

## `--role` (per-node service selection)

`start`, `stop`, `restart`, `status`, and `logs` all accept `--role`:

| Role | Manages |
|---|---|
| `all` (default) | API + plugin worker + protocol worker — identical to pre-`--role` behavior |
| `api` | FastAPI/uvicorn only |
| `plugins` | Plugin Celery worker only |
| `protocols` | Protocol Celery worker only |

This is the mechanism used to run only part of the stack on a given node in a [Multi-Node Cluster Deployment](../installation/multi-node-cluster.md) — for example, a compute node that should run the protocol worker but never the API:

```bash
./scripts/scipionapi runtime start --role protocols
./scripts/scipionapi runtime status --role protocols
./scripts/scipionapi runtime stop --role protocols
```

!!! note "Default behavior is unchanged"
    Omitting `--role` (or passing `--role all`) behaves exactly as before this option existed: all three processes are managed together.

An unrecognized role is rejected immediately with a clear error rather than silently falling back to `all`.

---

## `--queue` (custom Celery queue name)

`start`, `restart`, and `status` also accept `--queue`, which overrides the Celery queue name the **protocol** worker listens on (default: `protocols`). This is normally combined with `--role protocols` on a dedicated node:

```bash
./scripts/scipionapi runtime start --role protocols --queue protocols-gpu
```

`--queue` only affects that single invocation. To make the same queue name survive future restarts — including ones triggered from the web UI's worker restart button, which does not go through the CLI — set it in that node's `.env` instead:

```dotenv
PROTOCOLS_CELERY_QUEUE=protocols-gpu
```

A custom queue name only receives tasks once something is actually routed to it. See [Multi-Node Task Routing](../backend/celery.md#multi-node-task-routing) for how `scipion_home/config/celery_queue_routing.json` maps a protocol's configured Host to a Celery queue name.

!!! note "The worker's Celery identity does not change"
    Only the `-Q <queue>` flag changes. The worker still registers as `protocols@<hostname>`, so remote Stop and other host-targeted operations are unaffected by a custom queue name.

---

## Common issues

!!! warning "API starts but exits quickly"
    Check `logs` and confirm `.env` settings such as database, Valkey, and paths are valid.

!!! warning "Celery not running"
    Verify Valkey availability and broker configuration.

!!! warning "`status` says running but requests fail"
    Test runtime health explicitly with `/health` and inspect logs.

!!! warning "No logs shown"
    Confirm the configured logs path exists and the runtime user has write permissions.

---

## Current Runtime Worker Topology

The current `start`, `stop`, `restart`, and `status` implementation manages three runtime processes:

```text
ScipionAPI runtime
├── FastAPI / uvicorn
├── Plugin Celery worker
└── Protocol Celery worker
```

The two Celery workers consume different queues:

| Runtime process | Queue | Concurrency |
|---|---|---:|
| Plugin worker | `plugins` | `1` |
| Protocol worker | `protocols` | `PROTOCOL_WORKER_CONCURRENCY` (default `4`) |

When started by the CLI, the workers use the hostnames `plugins@%h` and `protocols@%h` and set `--prefetch-multiplier 1`.

The runtime also tracks the workers independently, so `status` can distinguish a healthy API from a missing plugin worker or protocol worker.

On a single node, all three processes normally run together (`--role all`, the default). On a [multi-node deployment](../installation/multi-node-cluster.md), each node manages only the subset relevant to it — for example a compute node running only `--role protocols`, optionally with its own `--queue` — while the master runs `--role api` and `--role plugins`.
