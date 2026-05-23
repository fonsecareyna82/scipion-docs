# Provision (One-Shot Installation)

The `provision` command performs a complete installation in a single step.

For most user-facing installations, the recommended setup is **Integrated Mode (API + Web)**. In this mode, ScipionAPI serves both the REST API and the compiled ScipionWeb interface from the same runtime.

It can:

- bootstrap the Conda environment
- install Python dependencies
- create or update `SCIPION_HOME`
- create PostgreSQL role and database in common local setups
- run Alembic migrations
- create or update the admin user
- deploy and serve the Web bundle
- start the API and Celery worker

!!! note "What this page covers"
    This page documents the **one-shot installation path** using `provision`. If you need fine-grained control over each step, use the **manual installation** workflow instead.

---

## Recommended mode: Integrated Mode (API + Web)

Integrated mode is the recommended path for normal ScipionWeb users because it provides a complete working application:

- Web UI served at `/`
- API mounted at `/api`
- API docs at `/api/docs`
- API and Web assets deployed together

This avoids asking users to install the backend without the web interface they actually need to use the platform.

Run the command from inside the extracted **ScipionAPI** directory and pass the compiled Web bundle ZIP with `--web-dist`:

```
./scripts/scipionapi provision \
  --user "admin" \
  --email "admin@example.com" \
  --web-dist "$HOME/scipionweb/ScipionWeb-<version>-dist.zip"
```

The CLI will ask for the admin password using a hidden prompt.

For automated installations, store the admin password in an environment variable and pass its name with `--password-env`:

```
./scripts/scipionapi provision \
  --user "admin" \
  --email "admin@example.com" \
  --password-env SCIPIONAPI_ADMIN_PASSWORD \
  --web-dist "$HOME/scipionweb/ScipionWeb-<version>-dist.zip"
```

!!! note "Password options"
    `--pass` and `--password` are still supported for compatibility, but `--password-env` or the hidden prompt are preferred because command-line arguments may be stored in shell history.

---

## API-only mode

API-only mode is available, but it is mainly useful for developers, infrastructure testing, or deployments where the frontend is hosted separately.

Use it only if you intentionally do **not** want ScipionAPI to serve the compiled Web UI:

```
./scripts/scipionapi provision \
  --user "admin" \
  --email "admin@example.com"
```

In API-only mode:

- the API is served directly by the backend
- the compiled Web UI is not deployed by this command
- users will need another frontend deployment to use ScipionWeb as an application

---

## When to choose `provision`

`provision` is the right default when:

- you want the fastest supported setup path
- you are installing on a local Linux machine
- PostgreSQL and Redis are already available or easy to prepare
- you want API and worker services started for you
- you want the Web UI deployed together with the API

If you want to inspect every installation layer manually, use `bootstrap` + `install` instead.

---

## What `provision` does

A typical integrated-mode run performs these steps in sequence:

1. prepares the Conda environment
2. installs Python dependencies
3. creates `SCIPION_HOME`
4. generates `.env`
5. prepares database state and migrations
6. creates or updates the admin user
7. deploys the Web bundle passed with `--web-dist`
8. starts API and Celery as detached services

---

## What to verify after it finishes

Run these checks immediately:

```
./scripts/scipionapi status
./scripts/scipionapi doctor --quick
./scripts/scipionapi logs
curl http://localhost:8080/health
```

For the recommended integrated mode, also verify that:

- `http://localhost:8080/` loads the Web UI
- `http://localhost:8080/api/docs` opens correctly

---

## Re-running `provision`

You may safely re-run `provision` in most common cases:

- it does **not** recreate the Conda environment if it already exists
- it does **not** drop existing databases
- it updates admin credentials when a password is provided
- it redeploys the Web bundle if `--web-dist` is provided again

That makes it useful after a failed first attempt once prerequisites are fixed.

---

## Common issues

!!! warning "Conda not found"
    Ensure `conda --version` works before running `provision`, or set `SCIPIONAPI_CONDA_EXE` to the Conda executable path.

!!! warning "PostgreSQL authentication failed"
    Verify the configured database credentials match the actual PostgreSQL role and password.

!!! warning "Redis not running"
    Start Redis and retry.

!!! warning "Web UI does not load after provisioning"
    Confirm that `--web-dist` pointed to the correct `ScipionWeb-<version>-dist.zip` file and that the API was restarted after deployment.

!!! warning "Port already in use"
    If port `8080` is already in use, stop the conflicting service or update your configuration before provisioning.

!!! tip "Use doctor for diagnostics"
    Run `./scripts/scipionapi doctor` to inspect Conda, `.env`, PostgreSQL, Redis, imports, runtime PID files, and Web deployment state.