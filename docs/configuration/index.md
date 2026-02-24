---
hide:
  - toc
---

# Configuration Overview

This section explains how **ScipionWeb** is configured at runtime and how configuration is organized across the system, runtime workspace, and application layers.

ScipionWeb configuration is centered around:

- `SCIPION_HOME`
- The `.env` file
- Deployment mode (API-only, integrated, or separate)
- Runtime environment variables and service connectivity

The configuration model is designed to:

- Keep runtime data separate from source code
- Allow flexible deployment (single host or distributed)
- Support production environments
- Enable reproducible installations and upgrades

!!! note "What this section covers"
    These pages focus on **runtime configuration** after installation. For installation and upgrade steps, see the **Installation** section.

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
    Keeping `SCIPION_HOME` separate from the API source/bundle directory makes upgrades easier and helps preserve data across version changes.

---

### `.env`

The `.env` file is the **main runtime configuration file**.

It typically defines:

- Database connection settings
- Redis / broker configuration
- API host and port
- Paths (logs, projects, web assets)
- Security-related values (for example `SECRET_KEY`)
- Deployment behavior flags (integrated mode, mount paths, etc.)

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

- Runtime directories and responsibilities
- Path conventions
- Permissions and persistence recommendations

➡️ [Open SCIPION_HOME and Runtime Layout](scipion-home/)

---

### 2. Environment Variables (`.env`)

Detailed reference for `.env` configuration values.

- Database settings
- API host/port
- Redis / broker settings
- Paths and runtime behavior
- Security-sensitive values

➡️ [Open Environment Variables (.env)](env/)

---

### 3. API + Web Integrated Mode

Configuration for serving the compiled Web UI from the API process.

- `SERVE_WEB`
- `WEB_DIST_PATH`
- API mount path / base URL alignment
- Routing expectations

➡️ [Open API + Web Integrated Mode](integrated-mode/)

---

### 4. Separate Deployment (API and Web on Different Hosts)

Configuration guidance for decoupled frontend/backend deployments.

- Cross-origin considerations
- API base URL setup
- Reverse proxy and public URL alignment
- Production deployment patterns

➡️ [Open Separate Deployment](separate-deployment/)

---

## Recommended Reading Paths

=== "First-time deployment (integrated mode)"

    1. [SCIPION_HOME and Runtime Layout](scipion-home/)
    2. [Environment Variables (.env)](env/)
    3. [API + Web Integrated Mode](integrated-mode/)

=== "Separate frontend/backend deployment"

    1. [SCIPION_HOME and Runtime Layout](scipion-home/)
    2. [Environment Variables (.env)](env/)
    3. [Separate Deployment (API and Web on Different Hosts)](separate-deployment/)

=== "Troubleshooting configuration"

    1. [Environment Variables (.env)](env/)
    2. [SCIPION_HOME and Runtime Layout](scipion-home/)
    3. Mode-specific page ([Integrated](integrated-mode/) or [Separate](separate-deployment/))

---

## Best Practices

- Never commit `.env` to version control
- Keep `SCIPION_HOME` outside the source repository (or at least outside tracked files)
- Use a strong `SECRET_KEY`
- Use HTTPS in production
- Use a reverse proxy for public deployments
- Back up runtime configuration before upgrades (especially `.env`)
- Keep API/Web versions aligned when possible

!!! warning "Common source of issues"
    Many runtime errors come from mismatched paths and URLs (for example, `WEB_API_BASE_URL`, API mount path, reverse proxy routes, or stale `SCIPION_HOME` values after upgrades).

---


<div style="display:flex; justify-content:space-between; align-items:center; width:100%; margin-top:2rem; gap:1rem;">
  <a href="../installation/" style="text-decoration:none; display:inline-block;">
    ← Previous: Installation Overview
  </a>
  <a href="scipion-home/" style="text-decoration:none; display:inline-block; margin-left:auto;">
    Next: SCIPION_HOME and Runtime Layout →
  </a>
</div>
