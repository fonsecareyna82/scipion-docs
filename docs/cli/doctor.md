---
hide:
  - toc
---

# `doctor` Command

The `doctor` command runs **read-only diagnostics** for a ScipionAPI installation.

Use it to verify the repository layout, Python environment, Conda setup, `.env` configuration, database connectivity, Valkey broker connectivity, imports, runtime PID files, and optional Web deployment state.

!!! tip "Safe to run anytime"
    `doctor` does not modify files, databases, Conda environments, or running services. It is designed for troubleshooting and validation.

---

## Usage

Run the default full diagnostics:

```
./scripts/scipionapi doctor
```

Run a faster check that skips heavier operations such as importing the FastAPI app and checking Alembic state:

```
./scripts/scipionapi doctor --quick
```

Run in strict mode, which exits with code `1` when failures are detected:

```
./scripts/scipionapi doctor --strict
```

You can combine both options:

```
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
- availability of commands such as `alembic`, `psql`, and `valkey-server`
- imports for key Python dependencies
- PostgreSQL connectivity using `DATABASE_URL`
- Valkey broker TCP connectivity using `BROKER_URL`
- API TCP reachability
- API and worker PID files
- integrated Web bundle layout when `SERVE_WEB=1`
- Alembic current revision in full mode
- multi-node deployment readiness (see below)

---

## Multi-Node Deployment Checks

`doctor` also validates configuration that only matters once more than one node participates in the same deployment (see [Multi-Node Cluster Deployment](../installation/multi-node-cluster.md)).

By default, `doctor` assumes a single-node deployment and only **warns** about the conditions below, so an ordinary local installation never fails these checks. Opt into strict multi-node validation with:

```bash
SCIPIONAPI_DEPLOYMENT_MODE=multi-node ./scripts/scipionapi doctor --strict
```

With `SCIPIONAPI_DEPLOYMENT_MODE=multi-node`, three checks escalate from `WARN` to `FAIL`:

| Check | What it verifies | Fails when (multi-node mode) |
|---|---|---|
| **Broker reachability** | `BROKER_URL` does not resolve to `localhost`/`127.0.0.1` | The broker host is a loopback address — unreachable from any other node |
| **Database reachability** | `DATABASE_URL` does not resolve to `localhost`/`127.0.0.1` | The database host is a loopback address — unreachable from any other node |
| **Projects filesystem** | `PROJECTS_PATH` is mounted from a filesystem type that looks shared (`nfs`, `nfs4`, `cifs`, `glusterfs`, `ceph`, ...) rather than local-only (`ext4`, `xfs`, `btrfs`, `tmpfs`, ...) | The filesystem type looks local-only, meaning other nodes would not see the same project/run files |

A fourth informational row, **Deployment mode**, always reports whether `SCIPIONAPI_DEPLOYMENT_MODE` is `single-node` (default) or `multi-node`.

!!! tip "Run this on every node"
    Run `SCIPIONAPI_DEPLOYMENT_MODE=multi-node ./scripts/scipionapi doctor --strict` on the master **and** on every worker node before starting services. It is the fastest way to catch a worker that would otherwise start, look healthy in its own log, and never actually receive any work because it is still pointed at its own `localhost` broker/database.

!!! note "Unset `SCIPIONAPI_DEPLOYMENT_MODE` is safe"
    Leaving the variable unset (or set to `single-node`) preserves the original behavior exactly: loopback broker/database URLs and a local filesystem for `PROJECTS_PATH` remain informational `WARN`s, never `FAIL`s.

---

## When to Use It

Run `doctor` after installation:

```
./scripts/scipionapi provision \
  --user "admin" \
  --email "admin@example.com"

./scripts/scipionapi doctor
```

Run it after starting services:

```
./scripts/scipionapi start
./scripts/scipionapi doctor --quick
```

Run it in automation or installer tests:

```
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

!!! warning "Valkey is unreachable"
    Start Valkey or update `BROKER_URL` to point to the correct broker.

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
