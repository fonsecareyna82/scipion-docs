---
hide:
  - toc
---

# Celery and Valkey

ScipionAPI uses **Celery** for background task execution.

Valkey acts as:

- **Broker**
- **Result backend**

This enables the backend to offload heavy or long-running work from the request/response path.

---

## Configuration

Defined in `.env`:

```dotenv
BROKER_URL=redis://localhost:6379/0
```

!!! note "Valkey role"
    In this backend architecture, Valkey is used by Celery infrastructure and must be available for task dispatching and processing.

---

## Worker Entry Point

Celery worker entry point:

```text
app/workers/task_queue.py
```

This module defines/loads the Celery app and task registration used by the worker process.

---

## Starting the Worker

### Via CLI (recommended for local usage)

```
./scripts/scipionapi start
```

This typically starts both:

- FastAPI/uvicorn
- Celery worker

### Manual worker start (advanced)

```
celery -A app.workers.task_queue worker --loglevel=info
```

!!! tip "Manual mode"
    Manual Celery startup is useful for debugging worker behavior independently from API startup.

---

## Typical Use Cases

Background tasks are useful for:

- Plugin installation
- Heavy computation tasks
- Long-running background jobs
- Operations that should not block API responses

---

## Task Flow

High-level task lifecycle:

```text
API → Celery Broker (Valkey) → Worker → Valkey (result/state) → API / client polling
```

!!! note "Response model"
    Depending on the API endpoint design, clients may receive immediate acknowledgement and then track progress/status asynchronously.

---

## Failure Handling

Operational behavior typically includes:

- Configurable retries (task-dependent)
- Error logging in worker logs (for example `celery.log`)
- Valkey connectivity dependency for enqueue/processing

!!! warning "Silent queueing assumptions"
    If the API is healthy but work is not progressing, confirm the worker is running and Valkey is reachable before debugging task code.

---

## Production Recommendations

- Run the worker via `systemd`
- Monitor Valkey health and memory usage
- Configure log rotation for worker logs
- Track worker restarts/failures
- Separate API and worker service supervision

!!! tip "Observability matters"
    Background-task failures are often operationally invisible to users until they inspect logs. Prioritize log visibility and service monitoring.

---

## Common Issues

!!! warning "Tasks are not processed"
    Check that the worker is running and connected to the same Valkey instance configured in `BROKER_URL`.

!!! warning "Worker starts but crashes"
    Inspect dependency imports, runtime environment activation, and backend logs (`celery.log`).

!!! warning "API enqueues task but no result appears"
    Verify Valkey connectivity, task registration, and result backend behavior.

!!! warning "Works locally, fails in production"
    Compare `.env`, systemd service environment, and filesystem permissions between environments.

---

## Quick Verification

```
./scripts/scipionapi status
./scripts/scipionapi logs
sudo systemctl status valkey-server
```

Look for:

- worker process running
- Valkey healthy
- task exceptions in `celery.log`

---

## Current Worker Topology

The current ScipionAPI runtime uses **two dedicated Celery workers** rather than a single generic worker. Tasks are routed to separate queues so plugin-management work and protocol execution can be controlled independently.

| Worker | Queue | Default concurrency | Hostname |
|---|---|---:|---|
| Plugin worker | `plugins` | `1` | `plugins@%h` |
| Protocol worker | `protocols` | `4` | `protocols@%h` |

The protocol-worker concurrency is configurable through `PROTOCOL_WORKER_CONCURRENCY`; the runtime default is `4`.

### Manual plugin worker

Run from the ScipionAPI repository root with the correct Python environment activated:

```bash
python -m celery -A app.workers.task_queue worker --loglevel info --hostname plugins@%h -Q plugins --concurrency 1 --prefetch-multiplier 1
```

### Manual protocol worker

In a separate terminal:

```bash
python -m celery -A app.workers.task_queue worker --loglevel info --hostname protocols@%h -Q protocols --concurrency 4 --prefetch-multiplier 1
```

For local development, keeping both workers in the foreground makes worker logs and task failures immediately visible.

### Task routing

Current routing separates plugin operations from protocol execution:

```text
plugins queue
├── app.tasks.installPluginTask
├── app.tasks.installPluginsBatchTask
├── app.tasks.installDevelPluginTask
└── app.tasks.uninstallPluginTask

protocols queue
└── app.tasks.executeProtocolTask
```

### Environment loading

Importing `app.workers.task_queue` loads the repository runtime environment from:

```text
scipion_home/.env
```

This allows manual worker commands to use the same Scipion runtime configuration as the backend without requiring a separate shell `source` of that file.

!!! note "Current Valkey configuration"
    Valkey is the supported Celery broker/result-backend server. Celery intentionally keeps the Redis-compatible `redis://` transport scheme. `BROKER_URL` configures the broker, while `RESULT_BACKEND_URL` optionally overrides the result backend and otherwise defaults to `BROKER_URL`.

### Verify both workers

```bash
python -m celery -A app.workers.task_queue inspect ping
```

A healthy local runtime should report both `plugins@...` and `protocols@...` workers.

---

## Navigation

<div style="display:flex; justify-content:space-between; align-items:center; width:100%; margin-top:2rem; gap:1rem;">
  <a href="../database/" style="text-decoration:none; display:inline-block;">
    ← Previous: Database and Alembic Migrations
  </a>
  <a href="../troubleshooting/" style="text-decoration:none; display:inline-block; margin-left:auto;">
    Next: Backend Troubleshooting →
  </a>
</div>
