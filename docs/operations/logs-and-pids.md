# Logs and PID Files

ScipionAPI runtime management relies on **log files** and **PID files** for monitoring, debugging, and service lifecycle operations.

---

## Log Location

Runtime logs are stored under:

```text
SCIPION_HOME/logs/
```

---

## Typical Log Files

```text
logs/
├── app.log
└── celery.log
```

### `app.log`

Usually contains API startup messages, request handling logs, and error traces.

### `celery.log`

Usually contains worker startup messages, task execution logs, failures, and retries.

!!! tip "First place to look"
    If the UI or API appears healthy but tasks fail silently, check `celery.log` early.

---

## Viewing Logs

### CLI shortcut

```
./scripts/scipionapi logs
```

### Manual live tail

```
tail -f scipion_home/logs/app.log
tail -f scipion_home/logs/celery.log
```

---

## PID Files

When using CLI runtime commands (`start`, `stop`, `restart`), ScipionAPI writes PID files under:

```text
.run/
```

Typical layout:

```text
.run/
├── api.pid
└── worker.pid
```

PID files help stop, restart, and inspect runtime services safely.

---

## Common pitfall: stale PID files

After an unexpected crash, PID files may remain even though the processes are gone.

Typical symptoms:

- `status` reports a process that is not really alive
- `start` refuses to launch because a PID file already exists
- `stop` cannot act on the recorded PID

Always verify the process state before assuming the PID file is trustworthy.

---

## How to use logs well

A useful operational habit is to correlate three things together:

1. the user action or failing workflow
2. the API log around that time
3. the worker log around that same time

This usually helps you decide whether the failure belongs to request handling, background execution, or environment state.

---

## Quick Ops Checklist

- [ ] `SCIPION_HOME/logs/` exists and is writable
- [ ] `app.log` updates during API requests
- [ ] `celery.log` updates during task execution
- [ ] `.run/` PID files match live processes
- [ ] logs are reviewed regularly in production

---

## Current Split-Worker Logs and PIDs

The current ScipionAPI runtime manages plugin and protocol workers independently. In addition to the files described above, the protocol worker has its own log and PID file.

Current runtime log layout:

```text
SCIPION_HOME/logs/
├── app.log
├── celery.log
└── celery-protocols.log
```

- `app.log` contains FastAPI/uvicorn output.
- `celery.log` contains the plugin worker output.
- `celery-protocols.log` contains the protocol worker output.

Current CLI-managed PID layout:

```text
.run/
├── api.pid
├── worker.pid
└── protocol-worker.pid
```

The files map to runtime processes as follows:

| PID file | Process |
|---|---|
| `api.pid` | FastAPI / uvicorn |
| `worker.pid` | Plugin Celery worker |
| `protocol-worker.pid` | Protocol Celery worker |

For manual log inspection, the protocol worker can be followed independently:

```bash
tail -f scipion_home/logs/celery-protocols.log
```

This separation is useful when API requests work correctly but only plugin tasks or protocol executions are failing.
