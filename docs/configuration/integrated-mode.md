---
hide:
  - toc
---

# API + Web Integrated Mode

Integrated mode allows **ScipionAPI** to serve the compiled **ScipionWeb** frontend directly, creating a **single deployment unit**.

In this model, the API process serves both:

- The Web UI (static frontend assets)
- The REST API
- API documentation endpoints

!!! note "Typical use case"
    Integrated mode is often the easiest option for **single-host deployments**, local installations, and internal lab servers where simplicity is more important than independent frontend scaling.

---

## Overview

In integrated mode:

- The Web UI is served at `/`
- The API is mounted under `/api`
- API docs are available under `/api/docs`

Typical local URLs:

```text
http://localhost:8080/
http://localhost:8080/api/docs
```

!!! tip "Single entry point"
    Users access one server and one base URL, which simplifies routing, networking, and operations.

---

## How It Works

Integrated mode is typically enabled during provisioning by passing the compiled Web bundle ZIP:

```bash
./scripts/scipionapi provision \
  --user "admin" \
  --email "admin@example.com" \
  --pass "changeMe" \
  --web-dist /path/to/web-dist.zip
```

During provisioning, the installer typically:

1. Extracts the Web `dist` bundle
2. Copies/deploys it into `SCIPION_HOME/web/dist`
3. Enables integrated mode (for example `SERVE_WEB=1`)
4. Mounts the API under `/api`

!!! note "Deployment path"
    The exact deployed location can vary by version/workflow, but integrated mode generally uses a runtime path under `SCIPION_HOME` to store the frontend assets.

---

## Runtime Behavior

Once integrated mode is enabled and the API is running:

- **Web UI** is served at `/`
- **API** is served at `/api`
- **API docs** are served at `/api/docs`

### Example endpoints

```text
http://localhost:8080/
http://localhost:8080/api/projects
http://localhost:8080/api/docs
```

!!! warning "Path consistency matters"
    The frontend must know the correct API base path. If the frontend expects `/api` but the API is mounted elsewhere, requests will fail.

---

## Required `.env` Variables

A typical integrated-mode configuration includes:

```dotenv
SERVE_WEB=1
API_MOUNT_PATH=/api
WEB_DIST_PATH=/path/to/dist
WEB_API_BASE_URL=/api
```

### Variable roles

- `SERVE_WEB=1` enables frontend serving from the API
- `API_MOUNT_PATH=/api` defines where the API is mounted
- `WEB_DIST_PATH` points to the compiled frontend `dist/` directory (or deployed assets path)
- `WEB_API_BASE_URL=/api` tells the frontend where to call the API

!!! tip "Keep these aligned"
    In most setups, `API_MOUNT_PATH` and `WEB_API_BASE_URL` should be the same value (usually `/api`).

---

## Typical Deployment Flow

=== "One-shot provisioning (recommended)"

    ```bash
    ./scripts/scipionapi provision \
      --user "admin" \
      --email "admin@example.com" \
      --pass "changeMe" \
      --web-dist "$HOME/scipionweb/ScipionWeb-<version>-dist.zip"
    ```

    This is the easiest path for a fresh local/server deployment.

=== "Manual integrated mode"

    1. Extract the Web bundle
    2. Set integrated-mode variables in `SCIPION_HOME/.env`
    3. Ensure `WEB_DIST_PATH` points to the deployed `dist/`
    4. Restart the API service

---

## When to Use Integrated Mode

Integrated mode is a good fit for:

- Simple deployments
- Single-host setups
- Local installations
- Internal research lab servers
- Environments where minimizing operational complexity is a priority

!!! tip "Great default for many labs"
    If you do not need CDN hosting, independent frontend deployments, or complex edge routing, integrated mode is often the most practical choice.

---

## Advantages

- **Single service endpoint** for users
- **No CORS issues** in the common case (same origin)
- **Simpler networking**
- **Easier installation and upgrades**
- **Fewer moving parts** for local/on-prem deployments

---

## Limitations

- Cannot scale frontend independently from the API
- Static frontend assets are served by the API process
- Less flexible for CDN-based frontend delivery
- Frontend updates are often tied to API deployment workflow

!!! note "Not a blocker for most small deployments"
    These limitations are usually acceptable for local, internal, or moderate-scale environments.

---

## Verification Checklist

After enabling integrated mode:

### 1. Check API health

```bash
curl http://localhost:8080/health
```

Expected response:

```json
{"status":"ok"}
```

### 2. Open the Web UI

Open:

```text
http://localhost:8080/
```

### 3. Open API docs

Open:

```text
http://localhost:8080/api/docs
```

### 4. Inspect browser requests (optional)

Confirm frontend API calls go to `/api/...` and not to an incorrect host/path.

---

## Common Issues

!!! warning "Frontend loads but API calls fail"
    Usually caused by a mismatch between `API_MOUNT_PATH` and `WEB_API_BASE_URL`, or a reverse proxy that rewrites paths incorrectly.

!!! warning "`WEB_DIST_PATH` points to old assets"
    After upgrades, verify that `WEB_DIST_PATH` references the current deployed frontend assets.

!!! warning "Got `/docs` instead of `/api/docs`"
    In integrated mode, docs are commonly served under the API mount path (for example `/api/docs`), not at `/docs`.

!!! warning "Blank page or missing static assets"
    Check that the frontend `dist/` contents were deployed correctly and that the API can read the files.

---

## Recommended Practices

- Keep API and Web versions aligned when possible
- Use integrated mode for simpler single-host deployments
- Re-check `.env` paths after upgrades
- Put a reverse proxy (nginx) in front for HTTPS/public access in production
- Back up `SCIPION_HOME/.env` before major changes

---


<div style="display:flex; justify-content:space-between; align-items:center; width:100%; margin-top:2rem; gap:1rem;">
  <a href="../env/" style="text-decoration:none; display:inline-block;">
    ← Previous: Environment Variables (.env)
  </a>
  <a href="../separate-deployment/" style="text-decoration:none; display:inline-block; margin-left:auto;">
    Next: Separate Deployment (API and Web on Different Hosts) →
  </a>
</div>
