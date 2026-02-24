---
hide:
  - toc
---

# Provision (One-Shot Installation)

The `provision` command performs a complete installation in a single step.

It can:

- Bootstrap the Conda environment
- Install Python dependencies
- Create or update `SCIPION_HOME`
- Create PostgreSQL role and database (local PostgreSQL)
- Run Alembic migrations
- Create or update the admin user
- Start the API and Celery worker
- Optionally deploy and serve the Web bundle

!!! note "What this page covers"
    This page documents the **one-shot installation path** using `provision`. If you need fine-grained control over each step, use the **manual installation** workflow instead.

---

## Recommended Usage

Run the command from inside the extracted **ScipionAPI** directory:

```bash
./scripts/scipionapi provision --user "admin" --email "admin@example.com" --pass "changeMe"
```

!!! warning "Credentials in shell history"
    Passing passwords directly on the command line may store them in your shell history. Use a temporary admin password for installation and change it afterwards if required.

---

## Integrated Mode (API + Web)

To serve the compiled Web UI together with the API, pass the Web bundle ZIP with `--web-dist`:

```bash
./scripts/scipionapi provision \
  --user "admin" \
  --email "admin@example.com" \
  --pass "changeMe" \
  --web-dist "$HOME/scipionweb/ScipionWeb-<version>-dist.zip"
```

This enables:

- Web UI served at `/`
- API mounted at `/api`
- API docs at `/api/docs`

!!! tip "No manual extraction required"
    You can pass the Web ZIP file directly. The installer handles deployment internally.

---

## What the `provision` Command Does

!!! info "Installation steps performed"
    1. Bootstraps the Conda environment (default: `scipion4Web`)
    2. Installs Python dependencies
    3. Creates `SCIPION_HOME` (default: `<repoRoot>/scipion_home`)
    4. Generates `.env`
    5. Creates local PostgreSQL role and database (when using local PostgreSQL)
    6. Applies `alembic upgrade head`
    7. Creates or updates the admin user
    8. Deploys the Web bundle (if `--web-dist` is provided)
    9. Starts API and Celery as detached services

---

## Integrated vs API-Only Modes

=== "API-only mode"

    ```bash
    ./scripts/scipionapi provision \
      --user "admin" \
      --email "admin@example.com" \
      --pass "changeMe"
    ```

    Access endpoints:

    - API docs: `http://localhost:8080/docs`

=== "Integrated mode"

    ```bash
    ./scripts/scipionapi provision \
      --user "admin" \
      --email "admin@example.com" \
      --pass "changeMe" \
      --web-dist /path/to/web-dist.zip
    ```

    Access endpoints:

    - Web UI: `http://localhost:8080/`
    - API docs: `http://localhost:8080/api/docs`

---

## Advanced Example

Use custom mount paths when deploying behind a reverse proxy or when you want explicit API path separation:

```bash
./scripts/scipionapi provision \
  --user "admin" \
  --email "admin@example.com" \
  --pass "changeMe" \
  --web-dist /path/to/web-dist.zip \
  --api-mount-path /api \
  --api-base-url /api
```

!!! tip "Path consistency"
    In integrated mode, `--api-mount-path` and `--api-base-url` should normally match to avoid frontend/API routing issues.

---

## After Provisioning

Useful operational commands:

### Check service status

```bash
./scripts/scipionapi status
```

### View logs

```bash
./scripts/scipionapi logs
```

### Restart services

```bash
./scripts/scipionapi restart
```

---

## Verify the Installation

### Check API health

```bash
curl http://localhost:8080/health
```

Expected response:

```json
{"status":"ok"}
```

### Check API docs

Open in your browser:

- **API-only mode:** `http://localhost:8080/docs`
- **Integrated mode:** `http://localhost:8080/api/docs`

!!! tip "If `/health` fails"
    Check service status and logs first (`status`, `logs`) before re-running `provision`.

---

## Common Issues

!!! warning "Conda not found"
    Ensure `conda --version` works before running `provision`.

!!! warning "PostgreSQL authentication failed"
    Verify the configured database credentials match the actual PostgreSQL role and password.

!!! warning "Redis not running"
    Start Redis and retry:
    ```bash
    sudo systemctl start redis-server
    ```

!!! warning "Alembic migration error"
    If migration history was modified manually, test with a **fresh empty database** before retrying.

!!! warning "Port already in use"
    If port `8080` is already in use, stop the conflicting service or update your configuration before provisioning.

---

## Re-running `provision`

You may safely re-run `provision` in most cases:

- It does **not** recreate the Conda environment if it already exists
- It does **not** drop existing databases
- It updates admin credentials if needed
- It redeploys the Web bundle if `--web-dist` is provided again

!!! note "Idempotent workflow (practical)"
    Re-running `provision` is useful after configuration changes, Web bundle updates, or failed first attempts once prerequisites are fixed.

---


<div style="display:flex; justify-content:space-between; align-items:center; width:100%; margin-top:2rem; gap:1rem;">
  <a href="../download-bundles/" style="text-decoration:none; display:inline-block;">
    ← Previous: Download Bundles (API + Web)
  </a>
  <a href="../manual-install/" style="text-decoration:none; display:inline-block; margin-left:auto;">
    Next: Manual Installation →
  </a>
</div>
