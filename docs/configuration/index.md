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

The configuration model is designed to:

- keep runtime data separate from source code
- allow flexible deployment (single host or distributed)
- support production environments
- enable reproducible installations and upgrades

!!! note "What this section covers"
    These pages focus on **runtime configuration after installation**. For installation and upgrade steps, see the **Installation** section.

---

## Configuration Layers

ScipionWeb configuration operates at three levels:

1. **System level** (Conda, PostgreSQL, Redis, OS services)
2. **Runtime level** (`SCIPION_HOME`)
3. **Application level** (`.env` variables and deployment mode)

!!! tip "Think in layers"
    When troubleshooting, identify the failing layer first:

    - service availability (system)
    - paths and workspace permissions (runtime)
    - environment variables and endpoints (application)

---

## Key Concepts

### `SCIPION_HOME`

`SCIPION_HOME` is the **runtime workspace directory**.

It stores persistent runtime data and configuration, typically including:

- `.env`
- `logs/`
- `projects/`
- `config/`
- `web/` *(if using integrated mode)*

!!! tip "Why this matters"
    Keeping `SCIPION_HOME` separate from the API source or bundle directory makes upgrades easier and helps preserve data across version changes.

---

### `.env`

The `.env` file is the **main runtime configuration file**.

It typically defines:

- database connection settings
- Redis and broker configuration
- API host and port
- paths (logs, projects, web assets)
- security-related values such as `SECRET_KEY`
- deployment behavior flags such as integrated mode and mount paths

!!! warning "Protect your `.env`"
    Do not commit `.env` to version control. It may contain credentials and security-sensitive settings.

---

### Deployment Mode

ScipionWeb supports multiple deployment modes:

- **API-only mode**
- **Integrated mode** (API serves the Web UI)
- **Separate deployment** (API and Web UI on different hosts or services)

Each mode changes how paths, URLs, and routing should be configured.

!!! note "Choose based on your environment"
    - **Integrated mode** is convenient for single-host deployments
    - **Separate deployment** is common for production setups with independent frontend hosting

---

## Default Behavior (Typical Local Setup)

If not overridden, a typical local installation uses:

- `SCIPION_HOME=<repoRoot>/scipion_home`
- API bound to `0.0.0.0:8080`
- Redis at `localhost:6379`
- PostgreSQL at `localhost:5432`

!!! tip "Defaults are a starting point"
    These defaults are convenient for local setups, but production deployments usually override at least paths, host exposure, and reverse-proxy behavior.

---

## Configuration Pages in This Section

### 1. `SCIPION_HOME` and Runtime Layout

Learn how the runtime workspace is structured and what should live inside `SCIPION_HOME`.

- runtime directories and responsibilities
- path conventions
- permissions and persistence recommendations

➡️ [Open SCIPION_HOME and Runtime Layout](scipion-home/)

---

### 2. Environment Variables (`.env`)

Detailed reference for `.env` configuration values.

- database settings
- API host and port
- Redis and broker settings
- paths and runtime behavior
- security-sensitive values

➡️ [Open Environment Variables (.env)](env/)

---

### 3. API + Web Integrated Mode

Configuration for serving the compiled Web UI from the API process.

- `SERVE_WEB`
- `WEB_DIST_PATH`
- API mount path and base URL alignment
- routing expectations

➡️ [Open API + Web Integrated Mode](integrated-mode/)

---

### 4. Separate Deployment (API and Web on Different Hosts)

Configuration guidance for decoupled frontend and backend deployments.

- cross-origin considerations
- API base URL setup
- reverse proxy and public URL alignment
- production deployment patterns

➡️ [Open Separate Deployment](separate-deployment/)

---

## Recommended Reading Paths

=== "First-time deployment (integrated mode)"

    1. [SCIPION_HOME and Runtime Layout](scipion-home/)
    2. [Environment Variables (.env)](env/)
    3. [API + Web Integrated Mode](integrated-mode/)
    4. [Logs and PID Files](../operations/logs-and-pids/)

=== "Separate frontend/backend deployment"

    1. [SCIPION_HOME and Runtime Layout](scipion-home/)
    2. [Environment Variables (.env)](env/)
    3. [Separate Deployment](separate-deployment/)
    4. [Security Notes](../operations/security/)

=== "Troubleshooting configuration"

    1. [Environment Variables (.env)](env/)
    2. [SCIPION_HOME and Runtime Layout](scipion-home/)
    3. mode-specific page ([Integrated](integrated-mode/) or [Separate](separate-deployment/))
    4. [Backend Troubleshooting](../backend/troubleshooting/)

---

## Best Practices

- never commit `.env` to version control
- keep `SCIPION_HOME` outside tracked source files when possible
- use a strong `SECRET_KEY`
- use HTTPS in production
- use a reverse proxy for public deployments
- back up runtime configuration before upgrades, especially `.env`
- keep API and Web versions aligned when possible

!!! warning "Common source of issues"
    Many runtime errors come from mismatched paths and URLs, for example `WEB_API_BASE_URL`, API mount path, reverse proxy routes, or stale `SCIPION_HOME` values after upgrades.

---

## Common configuration smells

Watch for these patterns:

- `.env` values that still point to an older installation path
- frontend assets deployed correctly but API base URL still wrong
- services running, but under a different `SCIPION_HOME` than expected
- local defaults accidentally reused in production

When behavior feels inconsistent, check paths and URLs before assuming code is broken.

---

<div style="display:flex; justify-content:space-between; align-items:center; width:100%; margin-top:2rem; gap:1rem;">
  <a href="../installation/" style="text-decoration:none; display:inline-block;">
    ← Previous: Installation Overview
  </a>
  <a href="scipion-home/" style="text-decoration:none; display:inline-block; margin-left:auto;">
    Next: SCIPION_HOME and Runtime Layout →
  </a>
</div>
