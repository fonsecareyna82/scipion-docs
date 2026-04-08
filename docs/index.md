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

=== "API-only mode"
    Use this mode when you want to host the frontend elsewhere or keep the backend standalone.

    - API docs: `http://host:8080/docs`

=== "Development mode"
    Run components independently:

    - React/Vite frontend: `http://localhost:5173`
    - FastAPI backend: `http://localhost:8080`

---

## Choose Your Path

### I want to install ScipionWeb for the first time

1. [Installation Overview](installation/)
2. [Prerequisites](installation/prerequisites/)
3. [Provision (One-Shot Installation)](installation/provision/)

### I want to deploy or operate it in a stable environment

1. [Configuration Overview](configuration/)
2. [Environment Variables (`.env`)](configuration/env/)
3. [Logs and PID Files](operations/logs-and-pids/)
4. [Security Notes](operations/security/)

### I am developing locally

1. [Local Dev Workflow](development/local-workflow/)
2. [Running Uvicorn Manually](development/uvicorn/)
3. [Common Errors and Fixes](development/common-errors/)

### I want to use the web application

1. [User Guide Overview](user-guide/)
2. [Projects](user-guide/projects/)
3. [Protocol Execution](user-guide/protocols/)
4. [Outputs and Viewers](user-guide/outputs/)
5. [Settings](user-guide/settings/)
6. [Plugins](user-guide/plugins/)
7. [Working Efficiently](user-guide/working-efficiently/)

---

## What You Get

!!! abstract "ScipionWeb Features"
    - **REST API** for projects, protocols, outputs, plugins, authentication, and sharing
    - **PostgreSQL persistence**
    - **Celery + Redis** background task execution
    - **Scipion-aware runtime integration**
    - **Optional integrated web serving**
    - **CLI tools** for installation, provisioning, runtime control, and logs

---

## Quick Example

```bash
./scripts/scipionapi provision \
  --user "admin" \
  --email "admin@example.com" \
  --pass "changeMe" \
  --web-dist /path/to/scipionweb-dist.zip
```
