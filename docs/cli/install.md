---
hide:
  - toc
---

# `install` Command

The `install` command configures the ScipionAPI **runtime workspace and database state** after Python/environment dependencies are already available.

It does not start runtime services. For a normal new ScipionWeb installation, prefer the public guided `install.sh`; use this command directly for advanced/manual setup.

---

## Usage

Interactive password prompt:

```bash
./scripts/scipionapi install \
  --user "admin" \
  --email "admin@example.org"
```

Automated form:

```bash
export SCIPIONAPI_ADMIN_PASSWORD='<admin-password>'

./scripts/scipionapi install \
  --user "admin" \
  --email "admin@example.org" \
  --password-env SCIPIONAPI_ADMIN_PASSWORD
```

!!! note "Password safety"
    `--pass` and `--password` remain available for compatibility, but hidden input or `--password-env` is preferred.

---

## Mental model

Use `install` when:

- the Conda/Python environment is already prepared
- you want to inspect configuration/database setup separately from startup
- you are debugging the lower-level provisioning sequence

Typical advanced sequence:

```text
bootstrap → install → doctor → start
```

`provision` combines those layers for convenience, and the guided `install.sh` adds host/release/download handling above `provision`.

---

## What it does

`install` typically:

1. resolves/creates `SCIPION_HOME`
2. creates or updates `SCIPION_HOME/.env`
3. creates runtime directories such as logs/projects
4. resolves the API/Web port
5. prepares the PostgreSQL role/database in supported local setups
6. applies Alembic migrations
7. creates or updates the administrator account

The administrator password is used for bootstrap/update of the account; it should not be treated as a persistent plaintext runtime setting.

---

## Port behavior

Use `--api-port` when you need a specific fixed port:

```bash
./scripts/scipionapi install \
  --user admin \
  --email admin@example.org \
  --api-port 39080
```

If `--api-port` is omitted:

- an existing `API_PORT` is preserved when appropriate
- otherwise a free port is selected automatically
- the resolved port is persisted in `.env`

Inspect it with:

```bash
grep '^API_PORT=' scipion_home/.env
```

Do not assume port `8080` for a newly configured installation.

---

## Database creation behavior

### Automatic local bootstrap

For a local PostgreSQL deployment with the supported administrative access, the installer can create/ensure the configured role and database automatically.

### Remote/custom PostgreSQL

For remote or custom PostgreSQL:

- prepare the database/role through the appropriate administrative process
- provide consistent connection settings
- verify connectivity before applying installation/migrations

Use the manual installation/configuration guidance for these advanced topologies.

---

## What to do next

```bash
./scripts/scipionapi doctor
./scripts/scipionapi start
./scripts/scipionapi status
```

Then resolve the actual port:

```bash
API_PORT="$(grep '^API_PORT=' scipion_home/.env | tail -n 1 | cut -d= -f2-)"
curl "http://localhost:${API_PORT}/health"
```

---

## Re-running `install`

Common installation operations are designed to be reusable:

- existing database data is not blindly dropped
- migrations are applied as needed
- administrator credentials can be updated
- existing API port is preserved unless intentionally changed

For an installed packaged release that needs a newer version, use `update` rather than treating the change as a fresh install.

---

## Common issues

!!! warning "PostgreSQL authentication failed"
    Verify the configured database endpoint/credentials and the actual PostgreSQL role.

!!! warning "Alembic migration error"
    Inspect database connectivity and migration state before retrying.

!!! warning "Port expectation mismatch"
    Read `API_PORT` from `.env`; automatic port selection is supported.

!!! warning "Environment looks inconsistent"
    Run `./scripts/scipionapi doctor` to inspect Conda, `.env`, PostgreSQL, Valkey, imports, and runtime state.
