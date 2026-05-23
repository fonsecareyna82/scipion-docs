---
hide:
  - toc
---

# `doctor` Command

The `doctor` command runs **read-only diagnostics** for a ScipionAPI installation.

Use it to verify the repository layout, Python environment, Conda setup, `.env` configuration, database connectivity, Redis broker connectivity, imports, runtime PID files, and optional Web deployment state.

!!! tip "Safe to run anytime"
    `doctor` does not modify files, databases, Conda environments, or running services. It is designed for troubleshooting and validation.

---

## Usage

Run the default full diagnostics:

```bash
./scripts/scipionapi doctor
```

Run a faster check that skips heavier operations such as importing the FastAPI app and checking Alembic state:

```bash
./scripts/scipionapi doctor --quick
```

Run in strict mode, which exits with code `1` when failures are detected:

```bash
./scripts/scipionapi doctor --strict
```

You can combine both options:

```bash
./scripts/scipionapi doctor --quick --strict
```

---

## What It Checks

`doctor` currently validates:

- repository markers such as `pyproject.toml`, `alembic.ini`, and the `app/` package
- Python version used by the CLI
- Conda executable resolution
- target Conda environment visibility
- active Conda environment
- `.env` presence and required variables
- `SCIPION_HOME`, logs, projects, and Scipion config files
- availability of commands such as `alembic`, `psql`, and `redis-server`
- imports for key Python dependencies
- PostgreSQL connectivity using `DATABASE_URL`
- Redis broker TCP connectivity using `BROKER_URL`
- API TCP reachability
- API and worker PID files
- integrated Web bundle layout when `SERVE_WEB=1`
- Alembic current revision in full mode

---

## When to Use It

Run `doctor` after installation:

```bash
./scripts/scipionapi provision \
  --user "admin" \
  --email "admin@example.com"

./scripts/scipionapi doctor
```

Run it after starting services:

```bash
./scripts/scipionapi start
./scripts/scipionapi doctor --quick
```

Run it in automation or installer tests:

```bash
./scripts/scipionapi doctor --quick --strict
```

---

## Understanding Results

| Status | Meaning |
|---|---|
| `OK` | The check passed |
| `WARN` | The check is not necessarily fatal, but should be reviewed |
| `FAIL` | The check failed and should be fixed |

A common pre-install result is a warning about the missing `.env` file. That is expected before running `install` or `provision`.

---

## Typical Issues Revealed by `doctor`

!!! warning "Wrong Conda environment"
    If the active environment differs from the expected one, run commands through `./scripts/scipionapi` or export `SCIPIONAPI_CONDA_ENV` explicitly.

!!! warning "PostgreSQL is unreachable"
    Check `DATABASE_URL`, local PostgreSQL service status, role permissions, and whether the target database exists.

!!! warning "Redis is unreachable"
    Start Redis or update `BROKER_URL` to point to the correct broker.

!!! warning "API TCP check failed"
    The services may not be running yet. Try `./scripts/scipionapi start` and then run `doctor` again.

---

## Navigation

<div style="display:flex; justify-content:space-between; align-items:center; width:100%; margin-top:2rem; gap:1rem;">
  <a href="../provision/" style="text-decoration:none; display:inline-block;">
    ← Previous: provision
  </a>
  <a href="../runtime/" style="text-decoration:none; display:inline-block; margin-left:auto;">
    Next: runtime commands →
  </a>
</div>
