---
hide:
  - toc
---

# Backend Troubleshooting

This page collects common backend issues and a practical debugging workflow for ScipionAPI.

!!! tip "Start with the basics"
    Most runtime issues are caused by one of these categories:

    - wrong Conda environment
    - invalid `.env` values
    - database connectivity problems
    - Redis / Celery runtime issues
    - CORS or auth misconfiguration

---

## Quick Triage Checklist

Before deep debugging:

```bash
./scripts/scipionapi status
./scripts/scipionapi logs
curl http://localhost:8080/health
```

Then verify:

- `SCIPION_HOME` points to the expected runtime workspace
- `.env` exists and has the expected values
- PostgreSQL and Redis are reachable

---

## 1. `ImportError`: `scipion` / `pyworkflow` not found

### Cause

- Not running inside the correct Conda environment

### Fix

```bash
conda activate scipion4Web
```

If using the wrapper script, re-run commands from the ScipionAPI bundle directory:

```bash
./scripts/scipionapi status
```

---

## 2. Database Connection Error

### Check

- PostgreSQL is running
- `DATABASE_URL` is correct
- Database role exists
- Network/firewall access is correct (remote DB setups)

### Test manually

```bash
psql -U user -d db
```

!!! warning "Credential mismatches"
    If `.env` was changed recently, ensure the running service was restarted and is not using stale environment values.

---

## 3. Alembic Migration Error

### Possible causes

- Inconsistent migration history
- Missing tables
- Modified migration files
- Wrong target database

### Debug steps

- Test on a fresh empty DB
- Inspect `alembic_version`
- Confirm migration files exist and match the deployed version

---

## 4. Redis Connection Error

Check Redis service status:

```bash
sudo systemctl status redis-server
```

Also verify the broker URL in `.env`:

```dotenv
BROKER_URL=redis://localhost:6379/0
```

---

## 5. Celery Worker Not Processing Tasks

### Check

- Worker is running
- Redis is reachable
- Worker logs (`celery.log`) show task receipt/execution
- `BROKER_URL` matches between API and worker runtime

### Useful commands

```bash
./scripts/scipionapi status
./scripts/scipionapi logs
```

---

## 6. CORS Errors

### Ensure

- Correct `allow_origins` in FastAPI app
- Proper frontend URL (scheme + host + port)
- Reverse proxy is not rewriting origins unexpectedly

!!! warning "Browser-only failures"
    If API requests work with `curl` but fail in the browser, CORS is a likely cause.

---

## 7. JWT Invalid Token

### Check

- `SECRET_KEY` matches the token-signing key
- Token is not expired
- `Authorization` header uses `Bearer <token>`
- User referenced by token `sub` still exists

---

## General Debugging Workflow

Enable verbose logging in development when possible, then inspect:

- `logs/app.log`
- `logs/celery.log`

Recommended sequence:

1. Stop services (if state is unclear)
2. Check logs
3. Validate `.env`
4. Validate DB connectivity
5. Validate Redis connectivity
6. Restart services
7. Re-test `/health` and failing endpoint

---

## When in Doubt (Recovery Loop)

```bash
./scripts/scipionapi stop
./scripts/scipionapi logs
./scripts/scipionapi start
./scripts/scipionapi status
curl http://localhost:8080/health
```

If the problem persists, isolate by subsystem:

- API startup only
- DB connectivity only
- Redis/Celery only
- Auth/CORS path only

---

## Escalation Tips for Teams

When reporting backend issues to teammates, include:

- exact command run
- error message (full traceback if available)
- relevant `.env` keys (redacted secrets)
- `status` output
- recent `app.log` / `celery.log` lines
- whether the issue reproduces on a fresh DB or staging

!!! tip "Faster debugging"
    A good bug report often saves more time than a quick patch attempt without logs.

---

## Navigation

<div style="display:flex; justify-content:space-between; align-items:center; width:100%; margin-top:2rem; gap:1rem;">
  <a href="../celery/" style="text-decoration:none; display:inline-block;">
    ← Previous: Celery and Redis
  </a>
  <span></span>
</div>
