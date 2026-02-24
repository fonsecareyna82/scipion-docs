---
hide:
  - toc
---

# Backend Overview

ScipionAPI is a **FastAPI-based backend** that exposes a REST API for:

- Project management
- Protocol execution
- Output previews
- Plugin management
- Authentication and authorization

It is designed to:

- Run inside a Scipion-capable Python environment
- Use PostgreSQL for persistence
- Use Redis for background task brokering
- Support both integrated and distributed deployments

!!! note "Scope of this section"
    This section documents the **backend architecture and runtime internals** (FastAPI, auth, database, Celery, and debugging). For installation and runtime configuration, see the **Installation** and **Configuration** sections.

---

## Core Responsibilities

The backend is responsible for:

- Managing users and authentication
- Managing projects and protocols
- Executing background tasks
- Serving output previews
- Integrating with the Scipion runtime
- Persisting metadata in PostgreSQL

---

## Technology Stack

- **API Framework**  
  **FastAPI** + **Uvicorn**

  Small, typed, high-performance HTTP layer for REST endpoints and OpenAPI docs.

- **Persistence**  
  **PostgreSQL**, **SQLAlchemy ORM**, **Alembic**

  Relational storage, ORM models, and versioned schema migrations.

- **Background Processing**  
  **Celery** + **Redis** *(broker / result backend)*

  Asynchronous task execution for long-running or heavy operations.

- **Security**  
  **JWT (HS256)** + **bcrypt**

  Token-based authentication and secure password hashing.
{: .grid .cards}

!!! tip "Why this layout"
    This version uses a card-friendly Markdown list (`{: .grid .cards}`), which is more reliable in MkDocs Material than mixing Markdown lists inside raw HTML blocks.

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

1. Loads environment from `SCIPION_HOME/.env`
2. Initializes the Scipion environment
3. Registers routers
4. Applies middleware and error handlers
5. Starts serving API requests

---

## Modes of Operation

ScipionAPI supports:

- **API-only mode**
- **Integrated mode** (API serves frontend)
- **Separate deployment** (API and Web on different hosts)

See the **Configuration** section for deployment-mode details.

---

## Backend Topics in This Section

### 1. Architecture

Layered design and request flow:

- Routers → Services → ORM / Mapper → Database
- Dependency injection boundaries
- Separation of concerns and extensibility

➡️ [Open Backend Architecture](architecture/)

### 2. FastAPI App and Routers

Application initialization and router registration:

- `main.py`
- CORS and error handlers
- Health endpoint and docs routes

➡️ [Open FastAPI App and Routers](fastapi/)

### 3. Authentication and JWT

Security model and token verification:

- JWT (HS256)
- `Authorization: Bearer <token>`
- `SECRET_KEY` and password hashing

➡️ [Open Authentication and JWT](auth/)

### 4. Database and Alembic Migrations

Persistence layer and schema evolution:

- PostgreSQL + SQLAlchemy
- Alembic migration flow
- Migration best practices

➡️ [Open Database and Alembic Migrations](database/)

### 5. Celery and Redis

Background tasks and worker runtime:

- Redis broker/backend
- Celery worker entry point
- Operational and production guidance

➡️ [Open Celery and Redis](celery/)

### 6. Backend Troubleshooting

Common backend failure modes and recovery steps:

- Import/runtime environment issues
- DB/Redis/Celery connectivity
- JWT/CORS problems
- Log-based debugging workflow

➡️ [Open Backend Troubleshooting](troubleshooting/)

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

=== "Security-focused review"

    1. [Authentication and JWT](auth/)
    2. [FastAPI App and Routers](fastapi/)
    3. [Backend Troubleshooting](troubleshooting/)

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
