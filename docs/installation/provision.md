# Provision (One-Shot Runtime Setup)

The `provision` command performs the complete ScipionAPI runtime setup in one step.

It remains the core installation engine, but for a normal new ScipionWeb installation you should usually use the public [Guided Installation](guided-install.md). The guided `install.sh` downloads the matching API/Web release and then delegates the actual environment, database, Web deployment, and startup work to `provision`.

Use `provision` directly when you intentionally manage the release bundles yourself or need more control over deployment options.

---

## What `provision` can do

It can:

- bootstrap the Conda environment
- install Python dependencies
- install Scipion core packages when needed
- create or update `SCIPION_HOME`
- create or update `.env`
- select or preserve the API/Web port
- create PostgreSQL role and database in common local setups
- run Alembic migrations
- create or update the admin user
- deploy and serve the Web bundle
- start the API and Celery worker

---

## Integrated mode: API + Web

For a manually downloaded release pair, run from the extracted **ScipionAPI** directory and pass the compiled Web ZIP:

```bash
./scripts/scipionapi provision \
  --user "admin" \
  --email "admin@example.com" \
  --web-dist "/path/to/ScipionWeb-v4.0.1-dist.zip"
```

The CLI asks for the administrator password using a hidden prompt.

For automation, use an environment variable:

```bash
export SCIPIONAPI_ADMIN_PASSWORD='<admin-password>'

./scripts/scipionapi provision \
  --user "admin" \
  --email "admin@example.com" \
  --password-env SCIPIONAPI_ADMIN_PASSWORD \
  --web-dist "/path/to/ScipionWeb-v4.0.1-dist.zip"
```

!!! note "Password options"
    `--pass` and `--password` remain available for compatibility, but a hidden prompt or `--password-env` is preferred because shell arguments can be stored in history or exposed through process inspection.

---

## Integrated-mode result

With a valid Web bundle, ScipionAPI configures an integrated deployment where:

- the Web UI is served from `/`
- the API is mounted under `/api` by default
- API documentation is exposed under the API mount path, typically `/api/docs`
- the Web build receives runtime API configuration rather than requiring a rebuild for each host

The deployed Web content is stored under the configured `SCIPION_HOME` runtime tree.

---

## API-only mode

API-only mode is useful for developers or deployments where the frontend is hosted elsewhere:

```bash
./scripts/scipionapi provision \
  --user "admin" \
  --email "admin@example.com"
```

In API-only mode, no compiled Web distribution is deployed by this command.

---

## Port behavior

The API/Web port is not assumed to be a fixed `8080`.

When `--api-port` is omitted:

1. an existing configured `API_PORT` is preserved when appropriate
2. otherwise a free port is selected automatically
3. the resolved value is persisted in `SCIPION_HOME/.env`

To force a specific port:

```bash
./scripts/scipionapi provision \
  --user "admin" \
  --email "admin@example.com" \
  --web-dist "/path/to/ScipionWeb-v4.0.1-dist.zip" \
  --api-port 39080
```

After provisioning, inspect the selected value:

```bash
grep '^API_PORT=' scipion_home/.env
```

---

## Useful advanced options

Examples of supported controls include:

```text
--web-dist PATH
--api-mount-path /api
--api-base-url URL
--api-port PORT
--bootstrap / --no-bootstrap
--env-name NAME
--python VERSION
--install-scipion-core / --no-install-scipion-core
--scipion-core-packages "..."
```

Use these options only when you have a deployment reason to override the defaults used by the guided installer.

---

## When to call `provision` directly

Direct `provision` is appropriate when:

- you downloaded or built the API/Web ZIPs yourself
- you are validating release artifacts
- you need API-only mode
- the frontend is hosted separately
- you are building custom deployment automation
- you need to customize the API mount path or runtime environment
- you are debugging a specific installation layer

For normal first-time installation, use [Guided Installation](guided-install.md).

---

## What happens during provisioning

A typical integrated-mode run performs:

1. Conda environment bootstrap when enabled
2. Python dependency installation
3. Scipion core installation when required
4. `SCIPION_HOME` and runtime-directory creation
5. `.env` generation/update
6. API/Web port resolution
7. PostgreSQL database/role preparation
8. Alembic migrations
9. admin-user creation/update
10. Web bundle deployment
11. API and Celery startup

---

## Verify after provisioning

Start with:

```bash
./scripts/scipionapi status
./scripts/scipionapi doctor --quick
```

Read the persisted port:

```bash
API_PORT="$(grep '^API_PORT=' scipion_home/.env | tail -n 1 | cut -d= -f2-)"
echo "$API_PORT"
```

Then test the API using that actual value:

```bash
curl "http://localhost:${API_PORT}/health"
```

For integrated mode, open:

```text
http://localhost:<API_PORT>/
http://localhost:<API_PORT>/api/docs
```

For deeper diagnostics:

```bash
./scripts/scipionapi doctor
./scripts/scipionapi logs
```

---

## Re-running `provision`

`provision` is designed to be reusable in common installation and recovery scenarios:

- the Conda environment is reused when already present
- existing database state is not blindly dropped
- admin credentials can be updated
- the Web bundle can be redeployed
- the configured API port is preserved unless intentionally changed

For an already-installed packaged release that simply needs a newer version, prefer `./scripts/scipionapi update` rather than treating the upgrade as a new provision.

---

## Common issues

!!! warning "Conda not found"
    Confirm the Conda executable or set `SCIPIONAPI_CONDA_EXE` before provisioning.

!!! warning "PostgreSQL administrative access failed"
    The standard local bootstrap expects working PostgreSQL plus sufficient privileges. Use the manual database path for custom/remote setups.

!!! warning "Valkey not responding"
    Verify `valkey-cli ping` returns `PONG`.

!!! warning "Web UI does not load"
    Confirm `--web-dist` points to the intended ScipionWeb release ZIP and inspect `status`, `doctor`, and `logs`.

!!! warning "Assuming port 8080"
    Read `API_PORT` from `.env` or the provisioning output. Automatic port selection is supported and should be expected.

---

## Relationship to the guided installer

The public installation path is intentionally layered:

> `install.sh` handles host checks + release resolution + downloads + checksum verification → `provision` handles runtime setup

That separation keeps the end-user experience simple while preserving a powerful direct CLI for advanced deployments.
