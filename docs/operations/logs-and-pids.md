---
hide:
  - toc
---

# Logs and PID Files

ScipionAPI runtime management relies on **log files** and **PID files** to support:

- Monitoring
- Debugging
- Service lifecycle operations
- Production stability

Understanding where these files live and how they are used makes incident response much faster.

---

## Log Location

Runtime logs are stored under:

```text
SCIPION_HOME/logs/
```

Default (if not overridden):

```text
<repoRoot>/scipion_home/logs/
```

---

## Typical Log Files

```text
logs/
├── app.log
└── celery.log
```

### `app.log`

Usually contains:

- API startup messages
- Request handling logs
- Error traces
- Scipion runtime messages

### `celery.log`

Usually contains:

- Worker startup messages
- Task execution logs
- Task failures
- Retry information

!!! tip "First place to look"
    If the UI or API appears healthy but tasks fail silently, check `celery.log` before changing code.

---

## Viewing Logs

### CLI shortcut

```bash
./scripts/scipionapi logs
```

### Manual live tail

```bash
tail -f scipion_home/logs/app.log
tail -f scipion_home/logs/celery.log
```

### Useful filters (optional)

```bash
grep -i "error" scipion_home/logs/app.log
grep -i "traceback" scipion_home/logs/celery.log
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

---

## What PID Files Are Used For

PID files store the process IDs of runtime services, typically:

- `uvicorn` (API)
- Celery worker

They are used to:

- Stop services safely
- Restart services
- Check current runtime state

---

## Check Running State

### Recommended

```bash
./scripts/scipionapi status
```

### Manual inspection

```bash
cat .run/api.pid
ps -p <pid>
```

You can do the same for the worker:

```bash
cat .run/worker.pid
ps -p <pid>
```

---

## Stale PID Files (Common After Crashes)

If a process exits unexpectedly, PID files may remain and report a false running state.

Symptoms:

- `status` says a service is running but it is not
- `start` refuses to launch due to existing PID file
- `stop` fails because PID is stale

### Cleanup stale PID files

```bash
rm -f .run/api.pid
rm -f .run/worker.pid
```

Then restart services:

```bash
./scripts/scipionapi start
```

!!! warning "Be careful in shared hosts"
    Only remove PID files after confirming the referenced processes are not actually running.

---

## Production Logging Strategy

Logs can grow quickly in production. Use one of these strategies:

- `logrotate`
- `systemd` + `journald`
- Centralized logging (ELK, Loki, Graylog, etc.)

### Example `logrotate` config

```text
/opt/scipionweb/runtime/logs/*.log {
    daily
    rotate 7
    compress
    missingok
    notifempty
}
```

!!! tip "systemd deployments"
    If you run API and Celery as `systemd` services, journald may be a better primary source for live operational logs.

---

## Best Practices

- Monitor logs regularly
- Rotate logs in production
- Alert on recurring errors
- Keep logs outside source code directories (via `SCIPION_HOME`)
- Validate PID behavior after unexpected restarts/crashes

---

## Quick Ops Checklist

- [ ] `SCIPION_HOME/logs/` exists and is writable
- [ ] `app.log` updates during API requests
- [ ] `celery.log` updates during task execution
- [ ] `.run/` PID files match live processes
- [ ] Log rotation is configured in production

---

## Navigation

<div style="display:flex; justify-content:space-between; align-items:center; width:100%; margin-top:2rem; gap:1rem;">
  <a href="../backup-restore/" style="text-decoration:none; display:inline-block;">
    ← Previous: Backup and Restore
  </a>
  <a href="../security/" style="text-decoration:none; display:inline-block; margin-left:auto;">
    Next: Security Notes →
  </a>
</div>
