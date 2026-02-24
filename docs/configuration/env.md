---
hide:
  - toc
---

# Environment Variables (`.env`)

The `.env` file is the **central runtime configuration file** for **ScipionAPI**.

It is typically stored at:

```text
SCIPION_HOME/.env
```

This file defines how the API connects to PostgreSQL and Redis, where runtime data is stored, how the API is exposed, and whether the Web UI is served in integrated mode.

!!! warning "Sensitive configuration"
    The `.env` file may contain credentials, secrets, and service endpoints. Do **not** commit it to version control, and protect file permissions in production.

---

## Overview

The `.env` file commonly contains configuration for:

- PostgreSQL database connection
- Security settings (`SECRET_KEY`)
- API runtime host and port
- Redis / Celery broker settings
- Filesystem paths (`logs`, `projects`)
- Integrated mode (API serving the Web UI)
- Conda integration (auto-detected during provisioning)
- Admin bootstrap values (used during install/provision workflows)

!!! note "Version-dependent variables"
    Some variables may be optional or version-specific. The exact set of supported keys can evolve across releases.

---

## File Location

Expected location:

```text
SCIPION_HOME/.env
```

Example:

```text
/opt/scipionweb/runtime/.env
```

!!! tip "Runtime separation"
    Keeping `.env` inside `SCIPION_HOME` (instead of the API bundle directory) makes upgrades safer and helps preserve configuration across bundle replacements.

---

## Core Database Variables

These variables configure the PostgreSQL connection.

```dotenv
DATABASE_URL=postgresql://user:password@host:5432/database
DATABASE_NAME=scipion_db
DATABASE_USER=scipion_user
DATABASE_PASS=securePassword
```

### Notes

- `DATABASE_URL` is typically the **authoritative connection string**
- `DATABASE_NAME`, `DATABASE_USER`, and `DATABASE_PASS` are useful for tooling, bootstrap logic, and operational scripts (depending on workflow)

!!! tip "Recommended practice"
    Keep the values consistent. A mismatch between `DATABASE_URL` and the split variables can cause confusing connection or bootstrap errors.

---

## Security

### `SECRET_KEY`

```dotenv
SECRET_KEY=generate_a_long_random_string
```

Used for:

- JWT token signing
- Authentication integrity

Requirements:

- **Strong**
- **Private**
- **Rotated when needed** (especially in production environments with security requirements)

!!! warning "Do not reuse weak secrets"
    Avoid short or guessable values (for example `secret`, `123456`, or repository placeholders).

---

## API Runtime

These variables define where the API listens.

```dotenv
API_HOST=0.0.0.0
API_PORT=8080
```

### Notes

- `0.0.0.0` binds the API to all network interfaces
- Change `API_PORT` if `8080` is already in use
- If using a reverse proxy (nginx), you may still keep the API on an internal port such as `8080`

!!! tip "Production exposure"
    For public deployments, prefer exposing the service through a reverse proxy with HTTPS instead of exposing the API directly on the internet.

---

## Celery / Redis

These variables configure the Celery broker/runtime behavior.

```dotenv
BROKER_URL=redis://localhost:6379/0
CELERY_LOGLEVEL=info
```

### Notes

- `BROKER_URL` points to the Redis instance used by Celery
- `CELERY_LOGLEVEL` controls worker log verbosity (for example `info`, `warning`, `debug`)

!!! warning "Redis availability"
    If Redis is down or unreachable, background task processing will fail even if the API starts successfully.

---

## Filesystem Paths

These variables define runtime storage locations used by ScipionAPI.

```dotenv
LOGS_PATH=scipion_home/logs
PROJECTS_PATH=scipion_home/projects
```

### Notes

- Paths may be relative (depending on how your runtime is structured) or absolute
- Ensure the service user has write permissions to these locations

!!! tip "Production recommendation"
    Use predictable, persistent paths and monitor disk usage—especially for `PROJECTS_PATH`, which can grow significantly over time.

---

## Integrated Mode Flags

These variables are used when the API also serves the compiled Web UI (**integrated mode**).

```dotenv
SERVE_WEB=1
API_MOUNT_PATH=/api
WEB_DIST_PATH=/absolute/path/to/dist
WEB_API_BASE_URL=/api
```

### Behavior

- `SERVE_WEB=1` enables integrated mode
- `SERVE_WEB=0` (or unset, depending on defaults) keeps the deployment in API-only mode
- `WEB_DIST_PATH` points to the extracted frontend `dist/` directory (or deployed web assets location)
- `API_MOUNT_PATH` and `WEB_API_BASE_URL` should usually match (for example both `/api`)

!!! warning "Routing mismatches"
    If `API_MOUNT_PATH` and `WEB_API_BASE_URL` do not align, the frontend may load but API calls can fail due to incorrect routes.

---

## Conda Integration

Provisioning and install workflows may auto-detect Conda and store related variables.

```dotenv
CONDA_EXE=/home/user/miniconda3/bin/conda
CONDA_ACTIVATION_CMD=...
```

### Notes

- These values may be auto-generated during provisioning
- Exact values can differ depending on shell, Conda installation path, and execution environment

!!! note "Do not guess activation commands"
    If you modify Conda-related variables manually, test the CLI workflow afterwards (`status`, `logs`, `restart`, etc.).

---

## Admin Bootstrap

Some workflows use admin bootstrap values during installation/provisioning.

```dotenv
ADMIN_USERNAME=admin
ADMIN_EMAIL=admin@example.com
ADMIN_PASSWORD=...
```

### Notes

- Used during install/provision bootstrap workflows
- May be read to create or update the initial admin user (depending on command and version)

!!! warning "Credentials handling"
    Do not keep placeholder passwords in production. If these values are used, protect `.env` permissions and rotate credentials after initial setup when appropriate.

---

## Example `.env` (Local / Integrated Mode)

This is an example composition using common values:

```dotenv
DATABASE_URL=postgresql://scipion_user:securePassword@localhost:5432/scipion_db
DATABASE_NAME=scipion_db
DATABASE_USER=scipion_user
DATABASE_PASS=securePassword

SECRET_KEY=replace_with_a_strong_random_secret

API_HOST=0.0.0.0
API_PORT=8080

BROKER_URL=redis://localhost:6379/0
CELERY_LOGLEVEL=info

LOGS_PATH=scipion_home/logs
PROJECTS_PATH=scipion_home/projects

SERVE_WEB=1
API_MOUNT_PATH=/api
WEB_DIST_PATH=/opt/scipionweb/runtime/web/dist
WEB_API_BASE_URL=/api
```

!!! tip "Keep examples separate from real secrets"
    Use documentation examples or template files for onboarding, but never store real production credentials in shared examples.

---

## Loading `.env` in Shell Sessions (Optional)

If you need to export `.env` variables into your current shell session for CLI tools:

```bash
set -a
source "$SCIPION_HOME/.env"
set +a
```

!!! note "Why `set -a`?"
    Many `.env` files use `KEY=value` format without `export`. `set -a` automatically exports variables loaded via `source`.

---

## Production Recommendations

- Do not expose `.env`
- Restrict file permissions (`chmod 600` or similarly restrictive, depending on deployment)
- Prefer HTTPS and a reverse proxy for public deployments
- Back up `.env` before upgrades
- Keep database values consistent across `DATABASE_URL` and split fields
- Review integrated mode paths after Web/API upgrades

!!! warning "Permission example"
    A common production setup is:

    ```bash
    chmod 600 "$SCIPION_HOME/.env"
    chown <service-user>:<service-user> "$SCIPION_HOME/.env"
    ```

---

## Common Mistakes

!!! warning "Stale `WEB_DIST_PATH`"
    After upgrading or moving the installation, `WEB_DIST_PATH` may still point to an old location.

!!! warning "Wrong database host/port"
    If PostgreSQL runs on another host or non-default port, `DATABASE_URL` must be updated accordingly.

!!! warning "Relative paths in unexpected working directories"
    Relative path values may behave differently depending on how the service is started (CLI, systemd, container, etc.).

!!! warning "Mismatched API base path"
    If you proxy the API under `/api`, confirm `API_MOUNT_PATH` and `WEB_API_BASE_URL` are set consistently.

---

## Quick Verification

Run a quick sanity check after editing `.env`:

```bash
test -f "$SCIPION_HOME/.env" && echo ".env found"
grep -E '^(DATABASE_URL|API_HOST|API_PORT|BROKER_URL|SERVE_WEB)=' "$SCIPION_HOME/.env" || true
```

Then verify the runtime:

```bash
curl http://localhost:8080/health
```

Expected response:

```json
{"status":"ok"}
```

---


<div style="display:flex; justify-content:space-between; align-items:center; width:100%; margin-top:2rem; gap:1rem;">
  <a href="../scipion-home/" style="text-decoration:none; display:inline-block;">
    ← Previous: SCIPION_HOME and Runtime Layout
  </a>
  <a href="../integrated-mode/" style="text-decoration:none; display:inline-block; margin-left:auto;">
    Next: API + Web Integrated Mode →
  </a>
</div>
