# Manual Installation

This guide describes a **controlled, step-by-step ScipionAPI installation** for advanced setups, development, and debugging.

For a normal new user-facing installation, use the [Guided Installation](guided-install.md) instead. The guided installer performs host checks, resolves the paired API/Web release, verifies checksums, and delegates runtime setup to `provision`.

Use the manual path when you need:

- remote/custom PostgreSQL setup
- explicit control over the Conda environment
- low-level migration/debugging work
- development environments
- custom deployment automation

---

## Mental model

Manual installation separates the layers that `provision` normally combines:

1. Python/Conda environment
2. database/runtime configuration
3. migrations and administrator creation
4. runtime startup and verification

That makes this path slower, but useful when you need to inspect each layer independently.

---

## 1. Prepare the ScipionAPI source/package

Start from an extracted ScipionAPI release bundle or a development checkout, depending on the task.

For a packaged release, verify that the root contains at least:

```text
app/
scipionapi_cli/
scripts/scipionapi
alembic/
pyproject.toml
alembic.ini
```

---

## 2. Create the Conda environment

Example development/manual environment:

```bash
conda create -n scipion4Web python=3.8 -y
conda activate scipion4Web
python -m pip install --upgrade pip
```

---

## 3. Install dependencies

```bash
pip install -r requirements.txt
pip install -e .
scipionapi --help
```

For packaged operational installs, prefer the project wrapper/bootstrap path where possible because it keeps Conda/runtime handling aligned with the normal ScipionAPI lifecycle.

---

## 4. Prepare PostgreSQL

For a custom/manual database setup, create the PostgreSQL role and database through your normal administrative process.

Make sure the values you create match the runtime configuration you will use later, especially:

```text
DATABASE_URL
DATABASE_NAME
DATABASE_USER
DATABASE_PASS
```

Verify connectivity before applying migrations.

---

## 5. Prepare `SCIPION_HOME`

Example local workspace:

```bash
mkdir -p scipion_home
export SCIPION_HOME="$(pwd)/scipion_home"
```

The persistent runtime configuration belongs at:

```text
$SCIPION_HOME/.env
```

Use the CLI's `install` command whenever possible to generate/align runtime state rather than hand-maintaining every key.

---

## 6. Configure/install runtime state

Use the hidden password prompt:

```bash
./scripts/scipionapi install \
  --user "admin" \
  --email "admin@example.org"
```

Or, for automation:

```bash
export SCIPIONAPI_ADMIN_PASSWORD='<admin-password>'

./scripts/scipionapi install \
  --user "admin" \
  --email "admin@example.org" \
  --password-env SCIPIONAPI_ADMIN_PASSWORD
```

This keeps administrator passwords out of ordinary shell arguments.

If you require a fixed API/Web port:

```bash
./scripts/scipionapi install \
  --user "admin" \
  --email "admin@example.org" \
  --api-port 39080
```

Otherwise the installation can preserve an existing `API_PORT` or select a free port automatically and persist it in `.env`.

---

## 7. Migrations

The supported `install` flow applies migrations as part of runtime setup.

When debugging Alembic manually, load the intended environment first:

```bash
set -a
source "$SCIPION_HOME/.env"
set +a
alembic upgrade head
```

Do this only when you intentionally need the lower-level migration step separate from `install`/`provision`.

---

## 8. Start services

```bash
./scripts/scipionapi start
./scripts/scipionapi status
```

Resolve the actual configured port:

```bash
API_PORT="$(grep '^API_PORT=' "$SCIPION_HOME/.env" | tail -n 1 | cut -d= -f2-)"
echo "$API_PORT"
```

Then verify health:

```bash
curl "http://localhost:${API_PORT}/health"
```

Do not assume port `8080` unless you explicitly configured that value.

---

## 9. Diagnostics

```bash
./scripts/scipionapi doctor --quick
./scripts/scipionapi doctor
./scripts/scipionapi logs
```

For development/manual work, keeping API and worker logs visible can make startup and queue problems easier to diagnose.

---

## Adding the Web UI manually

For an integrated deployment from a specific Web release bundle, the direct `provision` path is usually simpler than reproducing Web deployment by hand:

```bash
./scripts/scipionapi provision \
  --user admin \
  --email admin@example.org \
  --web-dist /path/to/ScipionWeb-vX.Y.Z-dist.zip
```

See [Provision](provision.md) and [Integrated Mode](../configuration/integrated-mode.md).

---

## When this path is best

Manual installation is appropriate for:

- development environments
- CI/testing
- remote PostgreSQL deployments
- migration/debugging investigations
- custom infrastructure
- release-package validation

For ordinary installations, the supported path remains:

```text
Prerequisites → install.sh --check-only → install.sh
```

---

## Common mistakes

!!! warning "Using a direct password argument"
    Prefer the hidden prompt or `--password-env` instead of putting real credentials in shell history.

!!! warning "Assuming port 8080"
    Read the persisted `API_PORT` from `.env` unless you explicitly chose a fixed port.

!!! warning "Database and `.env` disagree"
    Keep runtime database values aligned with the role/database you actually created.

!!! warning "Starting only one side of the runtime"
    Verify both API and Celery/Valkey behavior with `status`, `doctor`, and logs.

!!! warning "Using manual installation when a managed update is enough"
    For an existing packaged installation, use `./scripts/scipionapi update` for normal version changes.
