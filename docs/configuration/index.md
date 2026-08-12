---
hide:
  - toc
---

# Configuration Overview

This section explains how **ScipionWeb** is configured at runtime and how configuration is organized across the system, runtime workspace, and application layers.

ScipionWeb configuration is centered around:

- `SCIPION_HOME`
- the `.env` file
- deployment mode (API-only, integrated, or separate)
- runtime environment variables and service connectivity

The configuration model is designed to keep persistent runtime state separate from managed application files, support different deployment topologies, and preserve configuration across normal updates.

!!! note "What this section covers"
    These pages focus on **runtime configuration after installation**. For the recommended new-user path, start with the [Guided Installation](../installation/guided-install.md).

---

## Configuration layers

ScipionWeb configuration operates at three levels:

1. **System** — Conda, PostgreSQL, Redis, OS/network services
2. **Runtime workspace** — `SCIPION_HOME`
3. **Application** — `.env` values and deployment mode

!!! tip "Troubleshoot by layer"
    When something fails, first decide whether the problem is service availability, runtime paths/permissions, or application configuration.

---

## `SCIPION_HOME`

`SCIPION_HOME` is the persistent runtime workspace.

A standard guided installation uses:

```text
<installation-root>/scipion_home
```

It normally contains data/configuration such as:

```text
.env
logs/
projects/
config/
web/
```

The stable separation between managed application files and `SCIPION_HOME` is what allows normal updates to replace code/Web assets without treating projects and runtime configuration as release artifacts.

---

## `.env`

`SCIPION_HOME/.env` is the central persistent runtime configuration file.

It commonly defines:

- PostgreSQL connection/bootstrap values
- Redis/Celery settings
- API host and selected port
- logs/projects/Web paths
- `SECRET_KEY`
- integrated-mode flags and API mount paths
- Conda metadata
- update metadata

!!! warning "Protect `.env`"
    It can contain credentials and security-sensitive values. Do not commit it to version control.

---

## Deployment modes

ScipionWeb supports:

- **Integrated mode** — ScipionAPI serves both the Web UI and API
- **Separate deployment** — frontend and backend are hosted separately
- **API-only mode** — mainly for development, testing, or custom infrastructure

The guided installer targets integrated mode because it is the normal user-facing deployment.

---

## Typical local behavior

A normal local installation commonly uses:

```text
SCIPION_HOME=<installation-root>/scipion_home
PostgreSQL=localhost:5432
Redis=localhost:6379
```

The API/Web port is intentionally **not documented as a fixed `8080` default**.

When installation/provisioning is run without `--api-port`:

1. an existing configured `API_PORT` is preserved when appropriate
2. otherwise a free port is selected automatically
3. the resolved value is persisted in `.env`

Inspect it with:

```bash
grep '^API_PORT=' "$SCIPION_HOME/.env"
```

A production deployment can still intentionally configure a fixed internal port for systemd, nginx, Apache, containers, or firewall policy.

---

## Configuration pages

### `SCIPION_HOME` and Runtime Layout

Understand persistent runtime directories, ownership, and lifecycle.

➡️ [Open SCIPION_HOME and Runtime Layout](scipion-home.md)

### Environment Variables (`.env`)

Reference the persistent runtime configuration, including automatic port behavior.

➡️ [Open Environment Variables](env.md)

### API + Web Integrated Mode

Understand Web deployment, `/api`, runtime frontend configuration, and selected ports.

➡️ [Open Integrated Mode](integrated-mode.md)

### Separate Deployment

Configure frontend and backend as separate services/hosts.

➡️ [Open Separate Deployment](separate-deployment.md)

---

## Recommended reading paths

### Normal guided installation

1. [Guided Installation](../installation/guided-install.md)
2. [SCIPION_HOME](scipion-home.md)
3. [Environment Variables](env.md)
4. [Integrated Mode](integrated-mode.md)
5. [Logs and PID Files](../operations/logs-and-pids.md)

### Separate frontend/backend

1. [SCIPION_HOME](scipion-home.md)
2. [Environment Variables](env.md)
3. [Separate Deployment](separate-deployment.md)
4. [Security Notes](../operations/security.md)

### Troubleshooting

1. [Environment Variables](env.md)
2. [SCIPION_HOME](scipion-home.md)
3. deployment-mode page
4. [Backend Troubleshooting](../backend/troubleshooting.md)

---

## Best practices

- never commit `.env`
- preserve `SCIPION_HOME` during normal upgrades
- use strong secrets
- read the actual persisted `API_PORT` instead of assuming a port
- keep API/Web releases aligned in normal deployments
- use HTTPS/reverse proxying for public production exposure
- back up PostgreSQL and important configuration before risky maintenance
- prefer managed install/update/uninstall commands over manual filesystem deletion

---

## Common configuration smells

Watch for:

- `.env` values pointing to an old installation path
- a frontend deployed correctly but using the wrong API base path
- a service started under a different `SCIPION_HOME`
- operational scripts assuming port `8080` while the installation selected another port
- stale `WEB_DIST_PATH` after manual file moves
- inconsistent PostgreSQL values between `DATABASE_URL` and split settings

When behavior looks inconsistent, inspect the resolved runtime paths and `.env` before assuming a code defect.
