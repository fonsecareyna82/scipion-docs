---
hide:
  - toc
---

# Backend Overview

ScipionAPI is a **FastAPI-based backend** that exposes a REST API for:

- project management
- protocol execution
- output previews
- plugin management
- authentication and authorization

It is designed to:

- run inside a Scipion-capable Python environment
- use PostgreSQL for persistence
- use Redis for background task brokering
- support both integrated and distributed deployments

!!! note "Scope of this section"
    This section documents the **backend architecture and runtime internals**. For installation and runtime configuration, see the **Installation** and **Configuration** sections.

---

## Core Responsibilities

The backend is responsible for:

- managing users and authentication
- managing projects and protocols
- executing background tasks
- serving output previews
- integrating with the Scipion runtime
- persisting metadata in PostgreSQL

A practical mental model is that the backend is the operational center of ScipionWeb: the web UI requests data and actions, but persistence, orchestration, permissions, and asynchronous execution live here.

---

## Runtime Structure

Key modules (high-level):

```text
app/backend/
├── main.py
├── database.py
├── bootstrap.py
├── api/
│   ├── routers/
│   ├── services/
│   ├── schemas/
│   └── dependencies.py
├── models/
├── mapper/
├── utils/
```

!!! tip "Read this section in order"
    Start with **Architecture**, then **FastAPI**, **Auth**, **Database**, and **Celery**. Finish with **Troubleshooting** for operational debugging patterns.

---

## Startup Behavior (High Level)

At startup, ScipionAPI typically:

1. loads environment from `SCIPION_HOME/.env`
2. initializes the Scipion environment
3. registers routers
4. applies middleware and error handlers
5. starts serving API requests

If background execution is expected, a healthy Celery worker and Redis broker are also part of the effective runtime, even though they are not the same process as the API server.

---

## Recommended Reading Paths

=== "Backend developer onboarding"

    1. [Architecture](architecture/)
    2. [FastAPI App and Routers](fastapi/)
    3. [Database and Alembic Migrations](database/)
    4. [Authentication and JWT](auth/)
    5. [Celery and Redis](celery/)
    6. [Backend Troubleshooting](troubleshooting/)

=== "Ops / deployment debugging"

    1. [FastAPI App and Routers](fastapi/)
    2. [Database and Alembic Migrations](database/)
    3. [Celery and Redis](celery/)
    4. [Backend Troubleshooting](troubleshooting/)
    5. [Logs and PID Files](../operations/logs-and-pids/)

---

## Common backend failure boundaries

Many problems that feel like a single backend failure actually belong to different layers:

- API starts, but authentication fails
- API is healthy, but tasks never finish
- UI loads, but data is inconsistent
- migrations fail during upgrade

Identifying the layer first usually shortens debugging dramatically.

---

## Navigation

<div style="display:flex; justify-content:space-between; align-items:center; width:100%; margin-top:2rem; gap:1rem;">
  <a href="../cli/" style="text-decoration:none; display:inline-block;">
    ← Previous: CLI Reference
  </a>
  <a href="architecture/" style="text-decoration:none; display:inline-block; margin-left:auto;">
    Next: Backend Architecture →
  </a>
</div>
