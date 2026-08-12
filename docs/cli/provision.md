---
hide:
  - toc
---

# `provision` Command

`provision` performs the complete ScipionAPI runtime setup: bootstrap, installation/configuration, optional Web deployment, and service startup.

For a normal new end-user installation, prefer the public [Guided Installation](../installation/guided-install.md). The guided `install.sh` resolves/downloads the release and then delegates runtime setup to this command.

Use `provision` directly for manual bundle workflows, advanced deployments, development, and release validation.

---

## API-only usage

```bash
./scripts/scipionapi provision \
  --user "admin" \
  --email "admin@example.org"
```

The CLI asks for the admin password using hidden input.

For automation:

```bash
export SCIPIONAPI_ADMIN_PASSWORD='<admin-password>'

./scripts/scipionapi provision \
  --user "admin" \
  --email "admin@example.org" \
  --password-env SCIPIONAPI_ADMIN_PASSWORD
```

---

## Integrated API + Web usage

```bash
./scripts/scipionapi provision \
  --user "admin" \
  --email "admin@example.org" \
  --web-dist /path/to/ScipionWeb-vX.Y.Z-dist.zip
```

Automated form:

```bash
export SCIPIONAPI_ADMIN_PASSWORD='<admin-password>'

./scripts/scipionapi provision \
  --user "admin" \
  --email "admin@example.org" \
  --password-env SCIPIONAPI_ADMIN_PASSWORD \
  --web-dist /path/to/ScipionWeb-vX.Y.Z-dist.zip
```

!!! note "Password safety"
    `--pass` and `--password` are supported for compatibility, but hidden prompts or `--password-env` are preferred because command-line values can be exposed in shell history/process listings.

---

## Important options

| Option | Description |
|---|---|
| `--user` | Admin username |
| `--email` | Admin email |
| `--password-env` | Name of an environment variable containing the admin password |
| `--pass`, `--password` | Direct password argument; supported but not recommended |
| `--web-dist` | Vite `dist` directory or ZIP containing the compiled frontend |
| `--api-mount-path` | API mount path in integrated mode (default `/api`) |
| `--api-base-url` | API base URL written for the frontend |
| `--api-port` | Fixed API/Web port; if omitted, preserve existing `API_PORT` or select a free port |
| `--bootstrap`, `--no-bootstrap` | Enable/skip bootstrap phase |
| `--env-name` | Conda environment name |
| `--python` | Python version for the Conda environment |
| `--install-scipion-core`, `--no-install-scipion-core` | Control automatic Scipion core installation |
| `--scipion-core-packages` | Space-separated Scipion core package list |

Use `./scripts/scipionapi provision --help` for the current complete option list.

---

## What it does

A normal integrated run performs:

1. Conda bootstrap when enabled
2. Python dependency setup
3. Scipion core dependency setup when required
4. `SCIPION_HOME` creation/update
5. `.env` generation/update
6. API/Web port resolution
7. PostgreSQL role/database bootstrap in supported local setups
8. Alembic migrations
9. admin-user creation/update
10. Web bundle deployment
11. API and Celery startup

---

## Port selection

The runtime is not tied to port `8080`.

If `--api-port` is omitted, provisioning:

- preserves an existing configured `API_PORT` when appropriate, or
- selects a free port automatically

and persists the value in `SCIPION_HOME/.env`.

To choose a fixed port:

```bash
./scripts/scipionapi provision \
  --user admin \
  --email admin@example.org \
  --web-dist /path/to/ScipionWeb-vX.Y.Z-dist.zip \
  --api-port 39080
```

---

## Resulting URLs

Read the actual port after provisioning:

```bash
API_PORT="$(grep '^API_PORT=' scipion_home/.env | tail -n 1 | cut -d= -f2-)"
echo "$API_PORT"
```

### API-only mode

```text
http://localhost:<API_PORT>/docs
```

### Integrated mode

```text
http://localhost:<API_PORT>/
http://localhost:<API_PORT>/api/docs
```

---

## Verification

```bash
./scripts/scipionapi status
./scripts/scipionapi doctor --quick

API_PORT="$(grep '^API_PORT=' scipion_home/.env | tail -n 1 | cut -d= -f2-)"
curl "http://localhost:${API_PORT}/health"
```

For deeper diagnostics:

```bash
./scripts/scipionapi doctor
./scripts/scipionapi logs
```

---

## Re-running `provision`

Common provisioning operations are designed to be reusable:

- existing Conda environment can be reused
- database state is not blindly destroyed
- migrations are reapplied as needed
- admin credentials can be updated
- Web assets can be redeployed
- existing port configuration is preserved unless changed intentionally

For a normal version upgrade of an existing packaged installation, use `update` instead.

---

## When to use `provision` directly

Use it when:

- validating release ZIPs
- using a manual download workflow
- deploying API-only
- hosting the frontend separately
- customizing mount/base URLs
- debugging installation layers
- writing infrastructure automation

For ordinary new installations, start with `install.sh`.
