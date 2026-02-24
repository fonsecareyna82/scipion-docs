---
hide:
  - toc
---

# `provision` Command

The `provision` command performs a **complete one-shot installation** of ScipionAPI.

It combines:

- `bootstrap`
- `install`
- optional Web bundle deployment
- runtime service startup

!!! tip "Recommended for most users"
    `provision` is the fastest and most convenient way to get a working installation.

---

## Usage (API-only mode)

```bash
./scripts/scipionapi provision \
  --user "admin" \
  --email "admin@example.com" \
  --pass "changeMe"
```

---

## Usage (Integrated mode: API + Web)

```bash
./scripts/scipionapi provision \
  --user "admin" \
  --email "admin@example.com" \
  --pass "changeMe" \
  --web-dist /path/to/ScipionWeb-dist.zip
```

---

## Options

| Option | Description |
|---|---|
| `--user` | Admin username |
| `--email` | Admin email |
| `--pass` | Admin password |
| `--web-dist` | Path to compiled Web bundle (`.zip`) or extracted `dist/` folder |
| `--api-mount-path` | API mount path (default: `/api`) |
| `--api-base-url` | API base URL used by the frontend |

---

## What It Does

`provision` typically performs the following steps:

1. Bootstraps the Conda environment (if needed)
2. Installs Python dependencies
3. Configures the runtime workspace (`SCIPION_HOME`)
4. Creates the database/role (local PostgreSQL scenarios)
5. Runs Alembic migrations
6. Creates or updates the admin user
7. Deploys the Web bundle (optional)
8. Starts the API and Celery worker

---

## Resulting Access URLs

### API-only mode

```text
http://localhost:8080/docs
```

### Integrated mode

```text
http://localhost:8080/
http://localhost:8080/api/docs
```

---

## Re-running `provision`

`provision` is generally safe to run multiple times.

Typical behavior:

- Does **not** destroy the database
- Reapplies migrations (as needed)
- Updates admin credentials
- Replaces Web assets if `--web-dist` is provided again

!!! warning "Be intentional in production"
    Even idempotent commands should be run carefully in production. Confirm target paths, database endpoints, and Web bundle version before re-running.

---

## Common Usage Patterns

=== "First-time local setup"

    ```bash
    ./scripts/scipionapi provision \
      --user "admin" \
      --email "admin@example.com" \
      --pass "changeMe"
    ```

=== "Integrated mode with frontend bundle"

    ```bash
    ./scripts/scipionapi provision \
      --user "admin" \
      --email "admin@example.com" \
      --pass "changeMe" \
      --web-dist "$HOME/scipionweb/ScipionWeb-<version>-dist.zip"
    ```

=== "Custom API mount path"

    ```bash
    ./scripts/scipionapi provision \
      --user "admin" \
      --email "admin@example.com" \
      --pass "changeMe" \
      --web-dist /path/to/ScipionWeb-dist.zip \
      --api-mount-path /api \
      --api-base-url /api
    ```

---

## Verification After Provision

```bash
./scripts/scipionapi status
curl http://localhost:8080/health
```

Expected response:

```json
{"status":"ok"}
```

Then open:

- API-only mode: `http://localhost:8080/docs`
- Integrated mode: `http://localhost:8080/` and `http://localhost:8080/api/docs`

---

## Navigation

<div style="display:flex; justify-content:space-between; align-items:center; width:100%; margin-top:2rem; gap:1rem;">
  <a href="../install/" style="text-decoration:none; display:inline-block;">
    ← Previous: install
  </a>
  <a href="../runtime/" style="text-decoration:none; display:inline-block; margin-left:auto;">
    Next: runtime commands →
  </a>
</div>
