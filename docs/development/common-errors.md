# Common Errors and Fixes

This page collects frequent **development-time problems** and the fastest checks to resolve them.

!!! tip "Use this page for local dev issues"
    If your problem is more related to deployed services, production startup, or runtime operations, also check [Backend Troubleshooting](../backend/troubleshooting/).

---

## Quick Triage Checklist

Before going deep, verify the basics:

```bash
conda activate scipion4Web
./scripts/scipionapi status
curl http://localhost:8080/health
redis-cli ping
```

Then confirm:

- you are using the expected Conda environment
- `SCIPION_HOME` points to the right runtime workspace
- `.env` exists and contains the expected values
- PostgreSQL and Redis are reachable
- API and Celery are reading the same runtime configuration

---

## 1. `ImportError`: `scipion` or `pyworkflow` not found

### Typical cause

You are not running inside the expected Conda environment, or the worker was started outside the environment prepared by the wrapper script.

### Fix

```bash
conda activate scipion4Web
python -c "import pyworkflow; print(pyworkflow.__file__)"
```

If you are using the wrapper script, rerun the command from the ScipionAPI bundle directory:

```bash
./scripts/scipionapi restart
```

---

## 2. `DATABASE_URL` not set

### Symptom

```text
KeyError: DATABASE_URL
```

### Fix

- Ensure `SCIPION_HOME/.env` exists
- Ensure `SCIPION_HOME` is exported correctly
- Restart the API after changing environment values

Useful check:

```bash
echo "$SCIPION_HOME"
test -f "$SCIPION_HOME/.env" && echo ".env found"
```

---

## 3. Alembic migration conflicts

### Symptoms

- missing table errors
- duplicate revision errors
- multiple heads
- `Can't locate revision identified by ...`

### Fix

- inspect migration state with `alembic heads` and `alembic current`
- verify you are targeting the correct database
- avoid editing migrations that were already applied
- test on a fresh empty database when state is unclear

For real deployments, create a database backup before stamping or repairing migration state.

---

## 4. Redis connection refused

### Fix

```bash
sudo systemctl start redis-server
sudo systemctl status redis-server
redis-cli ping
```

Expected output:

```text
PONG
```

---

## 5. Celery is not processing tasks

### Check

- the worker is running
- Redis is reachable
- the broker URL matches between API and worker
- worker logs show task receipt/execution
- `SCIPION_HOME` is available to the worker process

### Useful commands

```bash
./scripts/scipionapi status
./scripts/scipionapi logs
redis-cli ping
```

If tasks stay in `PENDING`, confirm both the API and the worker are using the same runtime configuration.

---

## 6. CORS errors in the browser

### Symptoms

- requests work with `curl` but fail in the browser
- preflight or origin errors appear in DevTools

### Fix

- add the exact frontend origin to the backend CORS configuration
- verify scheme, host, and port match the real frontend URL
- check that reverse proxies are not rewriting origins unexpectedly

For local development, the browser origin is usually similar to:

```text
http://localhost:5173
```

The API may be running at:

```text
http://localhost:8080
```

Both values must match the configured development setup.

---

## 7. JWT invalid signature or invalid token

### Typical causes

- `SECRET_KEY` mismatch
- expired token
- stale token stored in the browser

### Fix

- ensure the API is using the expected `SECRET_KEY`
- clear stored tokens or log in again
- restart the backend if configuration changed recently

A private browser window is a quick way to test whether stale local state is involved.

---

## 8. Port already in use

### Symptom

```text
Address already in use
```

### Fix

Identify the process first:

```bash
lsof -i :8080
```

Then stop it only if it is safe to do so:

```bash
kill <pid>
```

Use forceful termination only as a last resort:

```bash
kill -9 <pid>
```

Or change the configured API port if another service is meant to keep using that port.

---

## 9. Permission denied inside `SCIPION_HOME`

### Fix

```bash
chmod -R u+rwX "$SCIPION_HOME"
```

Also confirm that the same user owns the runtime files and starts the services:

```bash
ls -ld "$SCIPION_HOME"
ls -ld "$SCIPION_HOME/logs"
```

---

## 10. When to escalate to backend troubleshooting

Go to [Backend Troubleshooting](../backend/troubleshooting/) when the issue involves:

- deployed services that do not start correctly
- database connectivity in shared or production environments
- Redis or Celery runtime mismatches
- authentication problems that reproduce outside local development

For recurring operational problems, also check [Known Issues and Workarounds](../support/known-issues/).

---

## Suggested Debugging Order

1. Check the active Conda environment
2. Validate `SCIPION_HOME` and `.env`
3. Test PostgreSQL and Redis reachability
4. Review API and Celery logs
5. Restart services and retry the failing workflow
