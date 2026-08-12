---
hide:
  - toc
---

# API + Web Integrated Mode

Integrated mode allows **ScipionAPI** to serve the compiled **ScipionWeb** frontend and REST API from one runtime endpoint.

This is the normal user-facing mode selected by the guided installer.

---

## Overview

In the standard integrated configuration:

- Web UI is served at `/`
- API is mounted under `/api`
- API docs are available under `/api/docs`

The host port is runtime configuration, not a fixed release property.

Read the actual port from the installed environment:

```bash
grep '^API_PORT=' "$SCIPION_HOME/.env"
```

Then the normal URLs are:

```text
http://localhost:<API_PORT>/
http://localhost:<API_PORT>/api/docs
```

!!! tip "Automatic port selection"
    When no fixed `--api-port` is supplied, provisioning can select a free port automatically and persist it in `.env`.

---

## Recommended installation path

For a new integrated installation:

```bash
wget https://scipion.cnb.csic.es/downloads/scipion/scipionWeb/install.sh
chmod +x install.sh
./install.sh --check-only
./install.sh
```

The installer resolves a paired API + Web release, verifies checksums, and delegates integrated runtime setup to `provision`.

---

## Direct/manual integrated provision

Advanced users can deploy a specific compiled Web artifact directly:

```bash
./scripts/scipionapi provision \
  --user "admin" \
  --email "admin@example.org" \
  --web-dist /path/to/ScipionWeb-vX.Y.Z-dist.zip
```

During provisioning, ScipionAPI deploys the Web assets under the runtime workspace and configures the frontend/API relationship.

---

## Runtime behavior

With integrated mode enabled:

```text
/             → compiled ScipionWeb frontend
/api/...      → REST API
/api/docs     → OpenAPI/Swagger documentation
```

Example using the real configured port:

```bash
API_PORT="$(grep '^API_PORT=' "$SCIPION_HOME/.env" | tail -n 1 | cut -d= -f2-)"

curl "http://localhost:${API_PORT}/health"
```

Open:

```text
http://localhost:<API_PORT>/
http://localhost:<API_PORT>/api/docs
```

---

## Important `.env` settings

A typical integrated installation includes:

```dotenv
SERVE_WEB=1
API_MOUNT_PATH=/api
WEB_DIST_PATH=/path/to/scipion_home/web/dist
WEB_API_BASE_URL=/api
API_PORT=<selected-port>
```

Roles:

- `SERVE_WEB=1` enables frontend serving
- `API_MOUNT_PATH` controls where the REST API is mounted
- `WEB_DIST_PATH` identifies the deployed frontend files
- `WEB_API_BASE_URL` tells the frontend where to call the API
- `API_PORT` is the persisted listening port

In the normal configuration, `API_MOUNT_PATH` and `WEB_API_BASE_URL` are both `/api`.

---

## Runtime frontend configuration

The compiled Web build is designed to be reusable across installations.

ScipionAPI injects/deploys runtime API configuration so the same Web release does not need to be rebuilt for every host or selected port.

This means:

- release ZIPs remain installation-independent
- automatic port selection does not require rebuilding ScipionWeb
- reverse proxies can be introduced without publishing a new frontend artifact

---

## When integrated mode is a good fit

Use it for:

- normal guided installations
- single-host deployments
- local/on-prem installations
- internal lab servers
- deployments where operational simplicity matters

Advantages include:

- same-origin API/UI traffic
- simple networking
- no separate static Web host requirement
- one update path for paired API + Web releases

---

## When to use a separate deployment

A separate frontend/backend topology may be appropriate when:

- Web assets are served by a CDN/static host
- API and frontend scale independently
- network/security zones require separation
- an existing infrastructure already provides static frontend hosting

See [Separate Deployment](separate-deployment.md).

---

## Verify integrated mode

### 1. Runtime status

```bash
./scripts/scipionapi status
./scripts/scipionapi doctor --quick
```

### 2. Resolve the actual port

```bash
API_PORT="$(grep '^API_PORT=' "$SCIPION_HOME/.env" | tail -n 1 | cut -d= -f2-)"
echo "$API_PORT"
```

### 3. Health endpoint

```bash
curl "http://localhost:${API_PORT}/health"
```

### 4. Browser

Open:

```text
http://localhost:<API_PORT>/
http://localhost:<API_PORT>/api/docs
```

### 5. Browser network requests

Confirm API requests use the expected `/api/...` base path.

---

## Common issues

!!! warning "Frontend loads but API calls fail"
    Check `API_MOUNT_PATH`, `WEB_API_BASE_URL`, and any reverse-proxy path rewriting.

!!! warning "Using the wrong port"
    Read `API_PORT` from `.env`; do not assume `8080` unless you explicitly configured it.

!!! warning "Stale Web assets"
    Use the managed update/deployment workflow so `WEB_DIST_PATH` points to the intended deployed release.

!!! warning "Opening `/docs` instead of `/api/docs`"
    In the standard integrated configuration, API documentation is under the API mount path.

---

## Updating integrated mode

For an installed system:

```bash
./scripts/scipionapi update --dry-run
./scripts/scipionapi update
```

The updater resolves the paired release from `manifest.json`, verifies checksums, updates managed API files, and redeploys the Web bundle while preserving `SCIPION_HOME`.
