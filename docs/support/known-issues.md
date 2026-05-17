# Known Issues and Workarounds

Use this page to document recurring problems that already have useful checks or workarounds.

This page is intentionally curated. GitHub Issues and Discussions may contain newer raw activity, but this page should summarize the patterns that appear often enough to help users recover faster.

---

## Celery tasks remain in `PENDING`

### Symptoms

- A plugin installation, protocol execution, or background action is submitted successfully.
- The task remains in `PENDING` for a long time.
- The UI does not show task progress.

### Common causes

- The Celery worker is not running.
- Redis is not reachable from the API or worker.
- The API and worker are using different `.env` files or different broker URLs.
- The worker started without the expected `SCIPION_HOME`.

### Checks

```bash
./scripts/scipionapi status
./scripts/scipionapi logs
redis-cli ping
```

Expected Redis response:

```text
PONG
```

### Workaround

Restart the runtime after verifying that `SCIPION_HOME`, `CELERY_BROKER_URL`, and `CELERY_RESULT_BACKEND` are consistent:

```bash
./scripts/scipionapi restart
```

---

## Redis reports no nodes replied within time constraint

### Symptoms

- Celery status commands fail or time out.
- Background tasks do not progress.
- Logs mention that no Redis nodes replied within the expected time.

### Common causes

- Redis is stopped or overloaded.
- The configured Redis URL points to the wrong host, port, or database.
- Firewall or network restrictions block access in a distributed deployment.

### Checks

```bash
sudo systemctl status redis-server
redis-cli ping
```

### Workaround

Start or restart Redis, then restart ScipionAPI services:

```bash
sudo systemctl restart redis-server
./scripts/scipionapi restart
```

---

## `SCIPION_HOME` is not available in the worker

### Symptoms

- The API starts, but Celery tasks fail immediately.
- Worker logs mention missing runtime configuration or missing `SCIPION_HOME`.
- Tasks behave differently when run from a shell and from the worker.

### Common causes

- The worker was started from a shell that did not export `SCIPION_HOME`.
- The worker process is not loading `SCIPION_HOME/.env`.
- API and worker were started through different mechanisms.

### Checks

```bash
echo "$SCIPION_HOME"
test -f "$SCIPION_HOME/.env" && echo ".env found"
./scripts/scipionapi logs
```

### Workaround

Start API and worker through the same runtime command so both processes share the same configuration source:

```bash
./scripts/scipionapi restart
```

---

## `ImportError`: `pyworkflow.project.manager` not found

### Symptoms

- Backend startup or Celery worker startup fails.
- Logs show an import error involving `pyworkflow` or `pyworkflow.project.manager`.

### Common causes

- The wrong Conda environment is active.
- Scipion dependencies were not installed in the environment used by the process.
- The worker is started outside the wrapper script or expected environment.

### Checks

```bash
conda activate scipion4Web
python -c "import pyworkflow; print(pyworkflow.__file__)"
```

### Workaround

Activate the expected environment and restart using the wrapper script from the ScipionAPI bundle directory:

```bash
conda activate scipion4Web
./scripts/scipionapi restart
```

---

## Browser CORS errors between frontend and backend

### Symptoms

- Requests work with `curl` but fail in the browser.
- Browser DevTools show CORS or preflight errors.
- This often appears in development mode with the Web UI on `localhost:5173` and the API on `localhost:8080`.

### Common causes

- The frontend origin is not allowed by the backend.
- The configured API URL does not match the real browser origin.
- A reverse proxy changes scheme, host, or port unexpectedly.

### Checks

- Confirm the exact frontend URL shown in the browser.
- Confirm the backend CORS configuration includes that exact origin.
- Check failed requests in browser DevTools.

### Workaround

Add the exact frontend origin to the backend CORS configuration and restart the API.

---

## Alembic cannot locate a revision

### Symptoms

- Migration commands fail with a message similar to `Can't locate revision identified by ...`.
- Upgrade or install fails while applying database migrations.

### Common causes

- The database migration history points to a revision that is not present in the current code checkout or bundle.
- The wrong database is being targeted.
- A partially applied migration state was left behind after a failed test or upgrade.

### Checks

```bash
alembic current
alembic heads
```

Also confirm that `DATABASE_URL` points to the intended database.

### Workaround

For test databases, the simplest fix is often to recreate the database and apply migrations from a clean state.

For real deployments, do not edit migration history blindly. Back up the database first, inspect the missing revision, and decide whether a controlled stamp or migration repair is appropriate.

---

## Port `8080` already in use

### Symptoms

- API startup fails with an address-in-use error.
- `provision` or `start` appears to fail immediately.

### Checks

```bash
lsof -i :8080
```

### Workaround

Stop the process using the port, or configure ScipionAPI to use a different API port.

Use forceful termination only as a last resort after confirming the process is safe to stop.

---

## When to add new entries

Add a new entry here when the same issue appears repeatedly in development, installation, user support, or GitHub discussions.

A useful entry should include symptoms, likely causes, checks, and a workaround.
