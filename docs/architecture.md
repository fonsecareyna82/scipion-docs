# Architecture

ScipionWeb is composed of three main layers:

- Web Frontend (React / Vite)
- FastAPI Backend (ScipionAPI)
- Infrastructure Layer (PostgreSQL + Redis + Filesystem)

---

# High-Level Architecture

```mermaid
flowchart LR
    Browser[User Browser]
    Web[ScipionWeb UI]
    API[FastAPI Backend]
    DB[(PostgreSQL)]
    Redis[(Redis)]
    FS[(SCIPION_HOME Filesystem)]

    Browser --> Web
    Web -->|HTTP /api| API
    API --> DB
    API --> Redis
    API --> FS
```

---

# Runtime Components

## FastAPI

- Exposes REST API
- Handles authentication (JWT)
- Serves preview endpoints
- Optionally serves compiled Web UI
- Initializes Scipion runtime environment

## Celery Worker

- Background tasks (plugin install, heavy operations)
- Uses Redis as broker + result backend

## PostgreSQL

Stores:

- Users
- Projects
- Protocols
- Project sharing
- Settings
- Tags

## Filesystem (SCIPION_HOME)

Contains:

- projects/
- logs/
- config/
- web/dist (if integrated mode)
- .env

---

# Integrated Mode

When `--web-dist` is provided during provisioning:

```mermaid
flowchart LR
    Browser --> API
    API -->|serves /| WebAssets
    API -->|mounted at /api| Routes
```

- Web is served at `/`
- API is mounted at `/api`
- API docs at `/api/docs`

---

# API-Only Mode

```mermaid
flowchart LR
    Browser --> WebServer
    WebServer -->|REST| API
    API --> DB
```

- Frontend hosted separately (e.g., nginx)
- API served independently
- Cross-origin requests enabled

---

# Startup Flow

```mermaid
sequenceDiagram
    participant User
    participant CLI
    participant Conda
    participant API
    participant DB

    User->>CLI: provision
    CLI->>Conda: create env (if needed)
    CLI->>DB: create role/database
    CLI->>API: run alembic upgrade
    CLI->>API: create admin
    CLI->>API: start uvicorn
    CLI->>API: start celery
```

---

# Port Usage

Default ports:

- API: 8080
- PostgreSQL: 5432
- Redis: 6379

---

# Security Model

- JWT authentication
- HS256 signature
- SECRET_KEY stored in `.env`
- Access tokens use `sub=email`

Production recommendation:

- Use HTTPS
- Reverse proxy (nginx)
- Rotate SECRET_KEY
