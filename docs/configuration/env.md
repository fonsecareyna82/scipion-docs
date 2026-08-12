---
hide:
  - toc
---

# Environment Variables (`.env`)

`SCIPION_HOME/.env` is the central persistent runtime configuration for ScipionAPI.

Typical location:

```text
<SCIPION_HOME>/.env
```

For a standard guided installation:

```text
<installation-root>/scipion_home/.env
```

The file stores runtime/service/database configuration that must survive managed application-code updates.

!!! warning "Sensitive configuration"
    `.env` can contain database credentials, secret keys, and service endpoints. Do not commit it to source control and protect its filesystem permissions.

---

## What is configured here

Common configuration groups include:

- PostgreSQL connection/bootstrap values
- application secret key
- API host and selected port
- Redis/Celery settings
- logs/projects/runtime paths
- integrated Web deployment settings
- Conda executable/activation metadata
- update metadata

The exact set evolves with ScipionAPI versions. Treat the installed CLI/code as authoritative for supported keys.

---

## Database settings

Typical values include:

```dotenv
DATABASE_URL=postgresql://user:password@host:5432/database
DATABASE_NAME=scipion_db
DATABASE_USER=scipion_user
DATABASE_PASS=<database-password>
```

Keep the split database values consistent with `DATABASE_URL` when both are present. Inconsistent values can make bootstrap, administration, and runtime connectivity behave differently.

---

## Security

A strong application secret is required:

```dotenv
SECRET_KEY=<strong-random-secret>
```

Use a long private value and protect the `.env` file from unauthorized reading.

---

## API runtime

Typical runtime keys include:

```dotenv
API_HOST=0.0.0.0
API_PORT=<selected-port>
```

### Automatic port selection

Do not assume that new installations always use port `8080`.

When `install`/`provision` is called without `--api-port`:

1. an existing configured `API_PORT` is preserved when appropriate
2. otherwise a free port is selected automatically
3. that value is persisted in `.env`

Read the actual value:

```bash
grep '^API_PORT=' "$SCIPION_HOME/.env"
```

If a fixed port is operationally required, pass `--api-port PORT` during installation/provisioning.

!!! tip "Reverse proxy deployments"
    The selected API port can remain an internal service port behind nginx/Apache or another reverse proxy. Public HTTPS exposure does not require hardcoding a particular backend port in the release bundle.

---

## Redis / Celery

Typical broker configuration:

```dotenv
BROKER_URL=redis://localhost:6379/0
CELERY_LOGLEVEL=info
```

Redis must remain reachable for Celery background processing.

---

## Runtime filesystem paths

Examples:

```dotenv
LOGS_PATH=/path/to/scipion_home/logs
PROJECTS_PATH=/path/to/scipion_home/projects
```

Depending on the installation version, paths may be absolute or resolved from the runtime workspace. Ensure the service user owns/has write access to the required runtime directories.

---

## Integrated Web mode

Integrated deployments commonly include:

```dotenv
SERVE_WEB=1
API_MOUNT_PATH=/api
WEB_DIST_PATH=/path/to/scipion_home/web/dist
WEB_API_BASE_URL=/api
```

Roles:

- `SERVE_WEB=1` enables frontend serving from ScipionAPI
- `API_MOUNT_PATH` controls where the REST API is mounted
- `WEB_DIST_PATH` identifies the deployed compiled frontend
- `WEB_API_BASE_URL` tells the Web UI which API base path/URL to use

For the normal integrated configuration, `API_MOUNT_PATH` and `WEB_API_BASE_URL` are usually both `/api`.

---

## Conda integration

Provisioning can persist Conda information such as:

```dotenv
CONDA_EXE=/home/user/miniconda3/bin/conda
CONDA_ACTIVATION_CMD=...
```

These values depend on the real Conda installation and should generally be managed by the provisioning/bootstrap flow rather than guessed manually.

---

## Administrator bootstrap metadata

The installation flow can persist non-secret administrator identity metadata when useful, for example username/email depending on release behavior.

Administrator passwords should be supplied through hidden prompts or temporary environment variables such as those used by the installer/CLI.

!!! warning "Do not rely on plaintext admin passwords in `.env`"
    The normal password flow uses the password to create/update the account. Do not add `ADMIN_PASSWORD=<real-password>` to `.env` as an operational shortcut.

---

## Update metadata

Successful updates can record metadata such as:

```dotenv
SCIPIONAPI_LAST_UPDATE_VERSION=v4.0.1
SCIPIONAPI_LAST_UPDATE_AT=20260812T143000Z
SCIPIONAPI_UPDATE_BASE_URL=https://scipion.cnb.csic.es/downloads/scipion/scipionWeb/
```

This helps the backend/UI describe installed release state and update availability.

---

## Example integrated `.env`

The exact generated values depend on the installation, but conceptually:

```dotenv
DATABASE_URL=postgresql://scipion_user:<database-password>@localhost:5432/scipion_db
DATABASE_NAME=scipion_db
DATABASE_USER=scipion_user
DATABASE_PASS=<database-password>

SECRET_KEY=<strong-random-secret>

API_HOST=0.0.0.0
API_PORT=39080

BROKER_URL=redis://localhost:6379/0
CELERY_LOGLEVEL=info

LOGS_PATH=/home/user/scipionweb/scipion_home/logs
PROJECTS_PATH=/home/user/scipionweb/scipion_home/projects

SERVE_WEB=1
API_MOUNT_PATH=/api
WEB_DIST_PATH=/home/user/scipionweb/scipion_home/web/dist
WEB_API_BASE_URL=/api
```

`39080` is only an example. Read the real `API_PORT` generated for your installation.

---

## Loading `.env` in a shell

When an administrative workflow genuinely needs the variables exported into the current shell:

```bash
set -a
source "$SCIPION_HOME/.env"
set +a
```

Be careful when doing this in shared shells because secrets become environment variables for child processes.

---

## Back up configuration

Before significant manual configuration or database changes:

```bash
cp "$SCIPION_HOME/.env" "$SCIPION_HOME/.env.backup"
```

For normal managed upgrades, the updater is designed to preserve the runtime workspace/configuration, but keeping an administrative backup remains sensible for important systems.

---

## Permissions

A common restrictive configuration is:

```bash
chmod 600 "$SCIPION_HOME/.env"
```

Ensure ownership matches the account that operates ScipionWeb.

---

## Common mistakes

!!! warning "Assuming port 8080"
    Resolve `API_PORT` from `.env` or runtime output. Automatic port selection is intentional.

!!! warning "Stale `WEB_DIST_PATH`"
    If Web assets are moved manually, integrated mode can point to old content. Normal update/deployment commands should manage the Web distribution path for you.

!!! warning "Mismatched API paths"
    In integrated mode, keep `API_MOUNT_PATH` and `WEB_API_BASE_URL` aligned unless a custom topology requires otherwise.

!!! warning "Database values disagree"
    Avoid inconsistent `DATABASE_URL`, database name, user, or password settings.

!!! warning "Editing generated Conda settings casually"
    Test `status`, `doctor`, and `restart` after any manual change to Conda/runtime configuration.

---

## Quick verification

```bash
test -f "$SCIPION_HOME/.env" && echo ".env found"
grep -E '^(DATABASE_URL|API_HOST|API_PORT|BROKER_URL|SERVE_WEB|API_MOUNT_PATH|WEB_DIST_PATH)=' \
  "$SCIPION_HOME/.env" || true
```

Then verify the runtime using the persisted port:

```bash
API_PORT="$(grep '^API_PORT=' "$SCIPION_HOME/.env" | tail -n 1 | cut -d= -f2-)"
curl "http://localhost:${API_PORT}/health"
```

Finally:

```bash
./scripts/scipionapi status
./scripts/scipionapi doctor --quick
```
