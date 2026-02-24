---
hide:
  - toc
---

# SCIPION_HOME and Runtime Layout

`SCIPION_HOME` is the **runtime workspace directory** used by **ScipionAPI**.

It stores runtime configuration, logs, project data, and (in integrated mode) deployed Web assets. This directory is part of your operational state and **must not be committed to version control**.

!!! note "Why `SCIPION_HOME` exists"
    `SCIPION_HOME` keeps runtime data **separate from the API source/bundle directory**, which makes upgrades safer and helps preserve data across version changes.

---

## Default Location

If `SCIPION_HOME` is not set explicitly, ScipionAPI typically uses:

```text
SCIPION_HOME=<repoRoot>/scipion_home
```

You can override it by exporting the variable before starting the application:

```bash
export SCIPION_HOME=/opt/scipionweb/runtime
```

!!! tip "Production recommendation"
    In production, use a dedicated path outside the API bundle directory (for example `/opt/scipionweb/runtime`) so upgrades do not affect runtime data.

---

## Typical Directory Structure

Example layout:

```text
scipion_home/
├── .env
├── logs/
│   ├── app.log
│   └── celery.log
├── projects/
│   └── <project folders>
├── config/
└── web/
    └── dist/
```

!!! note "Mode-dependent directories"
    The `web/` directory is typically present only in **integrated mode** (when the API serves the Web UI).

---

## What Each Path Is Used For

### `.env`

Primary runtime configuration file.

It typically contains:

- Database connection settings
- Security-related values (for example `SECRET_KEY`)
- Runtime paths
- Deployment mode flags
- API and broker configuration

!!! warning "Sensitive file"
    The `.env` file may contain credentials and secrets. Protect file permissions and never commit it to version control.

---

### `logs/`

Stores runtime logs, commonly including:

- API logs
- Celery worker logs

Typical uses:

- Debugging errors
- Operational monitoring
- Post-incident inspection

!!! tip "Log rotation"
    In production, plan a log rotation strategy (system-managed logs, `logrotate`, or journald-based logging depending on your deployment approach).

---

### `projects/`

Stores project runtime data, such as:

- Scipion projects
- Generated previews
- Runtime artifacts

This directory can grow significantly over time depending on project size and usage patterns.

!!! warning "Disk usage growth"
    Monitor available disk space and define a backup/retention strategy, especially for production or shared environments.

---

### `config/`

Optional location for Scipion-related runtime configuration files and generated configuration state, depending on your deployment and tooling.

!!! note "Contents may vary"
    The exact contents of `config/` may differ across versions or deployment workflows.

---

### `web/`

Present when using **integrated mode**.

Typical contents:

```text
web/dist/
    index.html
    assets/
    config.js
```

This directory stores the deployed frontend assets served by the API.

!!! tip "Web bundle upgrades"
    When upgrading the integrated Web UI, the deployed contents under `web/` may be replaced by a new `--web-dist` deployment.

---

## Permissions and Ownership

Ensure that the service user (or the user running ScipionAPI) has the correct access to `SCIPION_HOME`.

Minimum recommendations:

- **Read/write** access to `SCIPION_HOME`
- Write access to `logs/`, `projects/`, and deployed runtime paths
- Read access to `.env`
- Enough free disk space for project growth and logs

!!! warning "Permission mismatches"
    Avoid mixing `root`-owned files with services running as a non-root user. This is a common cause of startup failures and runtime write errors.

---

## Backup Strategy

A practical backup strategy should include:

- PostgreSQL database backup
- Entire `scipion_home/projects/` directory
- `SCIPION_HOME/.env`
- Any deployment-specific runtime configuration you rely on

!!! tip "Upgrade safety"
    Before upgrading ScipionAPI/ScipionWeb, back up both the database and key `SCIPION_HOME` contents so you can recover configuration and project data if needed.

---

## Production Recommendations

For production deployments, prefer a dedicated runtime path such as:

```text
/opt/scipionweb/runtime
```

instead of storing runtime data inside the API source or extracted bundle directory.

Recommended practices:

- Keep `SCIPION_HOME` outside version-controlled directories
- Protect `.env` permissions
- Monitor disk usage in `projects/` and `logs/`
- Back up runtime data regularly
- Reuse the same `SCIPION_HOME` across API bundle upgrades (when appropriate)

---

## Common Mistakes

!!! warning "Using a temporary directory"
    Do not place `SCIPION_HOME` under `/tmp` or other ephemeral locations. You may lose logs, configuration, and project data.

!!! warning "Forgetting to export `SCIPION_HOME` after upgrade"
    If you move to a new API bundle directory and forget to export the existing `SCIPION_HOME`, the application may create a new empty runtime workspace and appear misconfigured.

!!! warning "Committing runtime files"
    Avoid committing `scipion_home/` or `.env` files into Git repositories.

---

## Quick Verification

Run these checks to confirm the runtime workspace is correctly configured:

```bash
echo "$SCIPION_HOME"
ls -al "$SCIPION_HOME"
test -f "$SCIPION_HOME/.env" && echo ".env found"
test -d "$SCIPION_HOME/projects" && echo "projects/ found"
test -d "$SCIPION_HOME/logs" && echo "logs/ found"
```

!!! tip "If a path is missing"
    Some directories may be created lazily depending on how the application is used. The most critical item to verify early is `SCIPION_HOME/.env`.

---


<div style="display:flex; justify-content:space-between; align-items:center; width:100%; margin-top:2rem; gap:1rem;">
  <a href="../" style="text-decoration:none; display:inline-block;">
    ← Previous: Configuration Overview
  </a>
  <a href="../env/" style="text-decoration:none; display:inline-block; margin-left:auto;">
    Next: Environment Variables (.env) →
  </a>
</div>
