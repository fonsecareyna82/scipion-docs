---
hide:
  - toc
---

# FastAPI App and Routers

The FastAPI application is defined in:

```text
app/backend/main.py
```

This module is responsible for application initialization, middleware/error setup, router registration, and exposing the API runtime.

---

## Application Initialization

Main startup steps typically include:

1. Load environment via `bootstrapEnv()`
2. Create the FastAPI app instance
3. Register error handlers
4. Configure CORS
5. Prepare the Scipion runtime
6. Include routers

!!! note "Startup sequence"
    Exact ordering can vary slightly by version, but environment loading and core app initialization should happen before router usage.

---

## CORS Configuration

CORS is defined in `main.py`.

Example (development):

```python
allow_origins = [
    "http://localhost:5173",
    "http://localhost:5174"
]
```

Production guidance:

- Restrict origins to trusted frontend URLs
- Use HTTPS
- Avoid permissive wildcard origins unless strictly required for internal scenarios

!!! warning "Common production issue"
    Browser requests can fail due to CORS even when the API is healthy. Verify the **exact** frontend origin (scheme + host + port).

---

## Router Modules

Router modules are typically located in:

```text
app/backend/api/routers/
```

Common router groups include:

- `project_router`
- `protocol_router`
- `plugin_router`
- `auth_router`
- `user_router`

!!! tip "Router organization"
    Group endpoints by domain responsibility (projects, protocols, plugins, auth, users) to keep files focused and discoverable.

---

## Example Router Pattern

```python
@router.get("/projects")
def listProjects():
    return service.getProjects()
```

This illustrates the desired pattern:

- router defines HTTP endpoint
- service handles business logic

!!! note "Naming style"
    Function naming conventions may vary across the codebase. Keep naming consistent within each module and favor readability.

---

## Error Handling

Custom error handlers are registered via:

```text
registerAllErrorHandlers(app)
```

Typical handled categories:

- Validation errors
- Authentication errors
- Internal errors

### Why centralized handlers help

- Consistent API error responses
- Cleaner router code
- Easier logging and debugging
- Better client-side integration

---

## OpenAPI Docs

FastAPI documentation is available at:

```text
/docs
```

In integrated mode, it is commonly available at:

```text
/api/docs
```

*(actual final path depends on API mount configuration)*

!!! tip "Integrated mode reminder"
    If the API is mounted under `/api`, use `/api/docs` instead of `/docs`.

---

## Health Endpoint

Health endpoint:

```text
GET /health
```

Expected response:

```json
{"status":"ok"}
```

Use it for:

- runtime validation after startup
- monitoring checks
- deployment verification
- quick troubleshooting

---

## Operational Checks

After starting services:

```bash
./scripts/scipionapi status
curl http://localhost:8080/health
```

If requests fail in browser but `curl` works, inspect:

- CORS configuration
- reverse proxy path rewrites
- frontend API base URL settings

---

## Common Issues

!!! warning "Routers not reachable"
    Check that router modules are included in `main.py` and that the expected prefix/mount path is configured.

!!! warning "Docs route changed"
    In integrated mode or reverse-proxied setups, docs may be under `/api/docs` instead of `/docs`.

!!! warning "CORS errors in frontend only"
    Confirm `allow_origins` matches the real frontend origin exactly (scheme, domain, port).

!!! warning "Startup succeeds but runtime fails later"
    Validate database/Redis connectivity and inspect `logs/app.log`.

---

## Navigation

<div style="display:flex; justify-content:space-between; align-items:center; width:100%; margin-top:2rem; gap:1rem;">
  <a href="../architecture/" style="text-decoration:none; display:inline-block;">
    ← Previous: Backend Architecture
  </a>
  <a href="../auth/" style="text-decoration:none; display:inline-block; margin-left:auto;">
    Next: Authentication and JWT →
  </a>
</div>
