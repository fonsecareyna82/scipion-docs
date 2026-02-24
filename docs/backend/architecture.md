---
hide:
  - toc
---

# Backend Architecture

ScipionAPI follows a **layered architecture** designed to keep HTTP concerns, business logic, and persistence logic clearly separated.

Core flow:

```text
Routers → Services → ORM / Mapper → Database
```

!!! tip "Why this matters"
    This separation improves maintainability, testability, and feature evolution. Changes in one layer are less likely to leak across the whole backend.

---

## Layered Design

### Routers

Located in:

```text
app/backend/api/routers/
```

Responsibilities:

- Define HTTP endpoints
- Validate and parse requests
- Inject dependencies
- Return API responses

Routers should remain **thin** and focused on the HTTP layer.

---

### Services

Located in:

```text
app/backend/api/services/
```

Responsibilities:

- Business logic
- Project operations
- Protocol management
- Plugin orchestration
- Output preview generation

!!! note "Service boundary"
    Services should not depend on HTTP-specific details (status codes, request objects, etc.) unless a deliberate design choice requires it.

---

### Models (ORM)

Located in:

```text
app/backend/models/
```

Technology:

- SQLAlchemy ORM
- Declarative models
- Relationship mapping

Models represent persistent entities and relational structure used by the application.

---

### Mapper

The mapper layer provides additional relational operations beyond ORM convenience methods.

Typical uses:

- Project sharing
- Custom joins
- Flat relational queries
- Query patterns optimized for specific backend workflows

!!! note "ORM + mapper coexistence"
    Using both ORM models and mapper utilities is common when some queries are easier to express or maintain outside standard model methods.

---

## Request Flow

Typical request path:

```text
Client → Router → Dependency Injection → Service → DB → Response
```

### Expanded interpretation

1. **Client** sends HTTP request
2. **Router** matches endpoint and validates inputs
3. **Dependencies** provide user/session/context
4. **Service** executes business logic
5. **DB layer** persists/reads state
6. **Router** returns response payload

---

## Dependency Injection

FastAPI dependencies commonly provide:

- Current user
- JWT validation
- Database session

Defined in:

```text
api/dependencies.py
```

!!! tip "Strong dependency contracts"
    Keep dependency helpers focused and reusable (for example auth, DB session, permission checks). This keeps routers consistent and easier to audit.

---

## Separation of Concerns

ScipionAPI follows these practical boundaries:

- **Routers** do not contain business logic
- **Services** do not depend on the HTTP layer
- **Database/persistence** does not know about authentication policy details

### Why this is useful

- Easier testing (unit vs integration)
- Lower coupling between features
- Simpler refactors when APIs evolve
- Cleaner onboarding for new contributors

---

## Extensibility Pattern

A new feature typically requires:

1. New service method
2. New router endpoint
3. Optional database migration
4. Optional background task (Celery)

!!! tip "Feature checklist"
    When adding features, decide early whether the operation is:

    - synchronous (request/response), or
    - asynchronous (background task via Celery)

    That choice affects service design, API semantics, and user feedback flow.

---

## Example Change Path

=== "Simple read-only feature"

    1. Add service query method
    2. Add router endpoint
    3. Return serialized response

=== "State-changing feature"

    1. Add service method
    2. Add router endpoint
    3. Update ORM / mapper logic
    4. Add tests
    5. Add migration if schema changed

=== "Heavy/long-running feature"

    1. Add service entrypoint
    2. Enqueue Celery task
    3. Track task status / logs
    4. Return async-friendly API response

---

## Common Architecture Smells

!!! warning "Business logic inside routers"
    If routers grow large and contain branching domain logic, move that logic into services.

!!! warning "Service methods returning HTTP-specific responses"
    Returning raw HTTP response objects from services increases coupling to FastAPI/router concerns.

!!! warning "Direct DB usage scattered across routers"
    Keep persistence access centralized to services/mapper utilities where possible.

!!! warning "Hidden cross-layer dependencies"
    Be explicit about dependencies (auth, DB session, current user) to avoid brittle side effects.

---

## Navigation

<div style="display:flex; justify-content:space-between; align-items:center; width:100%; margin-top:2rem; gap:1rem;">
  <a href="../" style="text-decoration:none; display:inline-block;">
    ← Previous: Backend Overview
  </a>
  <a href="../fastapi/" style="text-decoration:none; display:inline-block; margin-left:auto;">
    Next: FastAPI App and Routers →
  </a>
</div>
