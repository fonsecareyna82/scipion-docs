---
hide:
  - toc
---

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

## `start`

Starts the API and Celery worker.

```bash
./scripts/scipionapi start
```

Typical behavior:

- Launches API and Celery as detached processes
- Creates PID files under `.run/`

---

## `stop`

Stops API and Celery.

```bash
./scripts/scipionapi stop
```

Typical behavior:

- Reads PID files
- Terminates the running processes (gracefully when possible)

!!! warning "Stale PID files"
    If a process was killed externally, PID files may become stale and require cleanup.

---

## `restart`

Stops and starts services again.

```bash
./scripts/scipionapi restart
```

Useful after:

- `.env` changes
- dependency/runtime updates
- integrated Web asset redeployments
- transient failures

---

## `status`

Shows whether services are running.

```bash
./scripts/scipionapi status
```

Example output:

```text
API: running (pid 1234)
Celery: running (pid 5678)
```

!!! note "Process state vs health"
    `status` confirms process state, but not necessarily application health. Also check logs and `/health`.

---

## `logs`

Tails application logs.

```bash
./scripts/scipionapi logs
```

This typically shows:

- `app.log`
- `celery.log`

### Log location

Defined by:

```dotenv
LOGS_PATH=scipion_home/logs
```

---

## Typical Runtime Workflow

```bash
./scripts/scipionapi start
./scripts/scipionapi status
./scripts/scipionapi logs
```

After configuration changes:

```bash
./scripts/scipionapi restart
./scripts/scipionapi status
```

Shutdown:

```bash
./scripts/scipionapi stop
```

---

## Production Note

For production deployments, consider:

- `systemd` services
- reverse proxy (nginx)
- dedicated logging/retention strategy

CLI runtime commands are ideal for:

- local development
- small internal setups
- debugging
- temporary/manual operations

---

## Common Issues

!!! warning "API starts but exits quickly"
    Check `./scripts/scipionapi logs` and confirm `.env` settings (database, Redis, paths) are valid.

!!! warning "Celery not running"
    Verify Redis availability and broker configuration (`BROKER_URL`).

!!! warning "`status` says running but requests fail"
    Test runtime health explicitly:

    ```bash
    curl http://localhost:8080/health
    ```

!!! warning "No logs shown"
    Confirm `LOGS_PATH` exists and the runtime user has write permissions.

---

## Navigation

<div style="display:flex; justify-content:space-between; align-items:center; width:100%; margin-top:2rem; gap:1rem;">
  <a href="../provision/" style="text-decoration:none; display:inline-block;">
    ← Previous: provision
  </a>
  <span></span>
</div>
