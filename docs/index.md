---
hide:
  - toc
---

# ScipionWeb Deployment Guide

!!! info "Overview"
    ScipionWeb combines a **FastAPI backend (ScipionAPI)** with an optional **compiled React frontend (ScipionWeb UI)** to provide a web interface for Scipion project and protocol management.

    This documentation explains how to install, configure, operate, and use ScipionWeb in different modes.

## Deployment Modes

=== "Integrated mode (recommended)"
    The backend serves the compiled frontend and mounts the API under `/api`.

    - Web UI: `http://host:8080/`
    - API docs: `http://host:8080/api/docs`

    !!! tip
        Enable this mode during provisioning by passing a compiled frontend bundle with `--web-dist`.

=== "API-only mode"
    Use this mode when you want to host the frontend elsewhere or keep the backend standalone.

    - API docs: `http://host:8080/docs`

=== "Development mode"
    Run components independently:

    - React/Vite frontend: `http://localhost:5173`
    - FastAPI backend: `http://localhost:8080`

    !!! note
        This is the preferred mode for active development and debugging.

---

## What You Get

!!! abstract "ScipionWeb Features"
    - **REST API** for projects, protocols, outputs, plugins, authentication, and sharing
    - **PostgreSQL persistence**
    - **Celery + Redis** background task execution
    - **Scipion-aware runtime integration**
    - **Optional integrated web serving** (compiled React frontend mounted by the backend)
    - **CLI tools** for installation, provisioning, runtime control, and logs

---

## Recommended Installation Model

!!! success "Bundle-based deployment (recommended)"
    1. Download the **ScipionAPI bundle** (server)
    2. Download the **ScipionWeb bundle** (compiled web UI, optional for integrated mode)
    3. Extract both bundles
    4. Run a single provisioning command

---

## High-level Install Flow

!!! example "Typical first-time installation on Linux"
    1. Install prerequisites:
        - Conda (Miniconda or Anaconda)
        - PostgreSQL
        - Redis
    2. Download ScipionAPI and (optionally) ScipionWeb compiled bundle
    3. Extract bundles
    4. Run provisioning:
        - creates the Conda environment (if missing)
        - installs Python dependencies
        - generates `SCIPION_HOME/.env`
        - creates local PostgreSQL role/database (if configured for local PostgreSQL)
        - runs Alembic migrations
        - creates/updates admin user
        - starts API and Celery worker
        - optionally deploys and serves the compiled web UI

---

## Quick Example

```bash
./scripts/scipionapi provision \
  --user "admin" \
  --email "admin@example.com" \
  --pass "changeMe" \
  --web-dist /path/to/scipionweb-dist.zip
```

---

## Choose Your Path

Use the route that best matches your role:

### I want to install ScipionWeb for the first time

Start here:

1. [Installation Overview](installation/)
2. [Prerequisites](installation/prerequisites/)
3. [Provision (One-Shot Installation)](installation/provision/)

### I want to deploy or operate it in a stable environment

Start here:

1. [Configuration Overview](configuration/)
2. [Environment Variables (`.env`)](configuration/env/)
3. [Logs and PID Files](operations/logs-and-pids/)
4. [Security Notes](operations/security/)

### I am developing locally

Start here:

1. [Local Dev Workflow](development/local-workflow/)
2. [Running Uvicorn Manually](development/uvicorn/)
3. [Common Errors and Fixes](development/common-errors/)

### I want to use the web application

Start here:

1. [User Guide Overview](user-guide/)
2. [Projects](user-guide/projects/)
3. [Protocol Execution](user-guide/protocols/)
4. [Outputs and Viewers](user-guide/outputs/)

---
