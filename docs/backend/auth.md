---
hide:
  - toc
---

# Authentication and JWT

ScipionAPI uses **JWT-based authentication** for API access control.

Algorithm:

```text
HS256
```

---

## Token Model

Common token concepts:

- **Access token**
- **Refresh token** *(if implemented in your deployment/version)*
- `sub` claim containing the user email

!!! note "Version-dependent behavior"
    Refresh-token support may be optional or evolve over time. Confirm the exact flow implemented in your backend version.

---

## Login Flow

Typical authentication flow:

1. User submits credentials
2. Password is verified against the stored hash
3. JWT is generated
4. Token is returned to the client

---

## Authorization Header

Clients must send:

```text
Authorization: Bearer <token>
```

This header is typically included on protected API requests.

!!! warning "Common client error"
    Missing `Bearer` prefix or malformed headers will cause authentication failures even if the token string itself is valid.

---

## JWT Verification

JWT verification is performed in:

```text
api/dependencies.py
```

Typical verification steps:

1. Decode token using `SECRET_KEY`
2. Validate signature
3. Extract `sub`
4. Fetch user from database

!!! tip "Dependency-based auth"
    Performing verification in dependencies keeps router code clean and standardizes auth handling across endpoints.

---

## `SECRET_KEY`

Defined in `.env`:

```dotenv
SECRET_KEY=...
```

Requirements:

- Strong
- Private
- Rotated if compromised

!!! warning "Critical secret"
    If `SECRET_KEY` is exposed, existing tokens may become untrusted. Rotate the key and re-issue credentials/tokens according to your security policy.

---

## Password Hashing

Passwords are stored **hashed** (bcrypt).

- Never stored in plain text
- Verified by comparing the submitted password against the stored hash

!!! note "Security hygiene"
    Keep hashing and verification logic centralized and avoid duplicating password-processing code across modules.

---

## Production Recommendations

- Use HTTPS everywhere
- Restrict token lifetime
- Rotate `SECRET_KEY` periodically (planned process)
- Implement refresh tokens if required by your UX/security model
- Audit CORS + auth behavior together in browser-based deployments

---

## Common Auth Failures

!!! warning "Invalid token"
    Check token expiry, signature validity, and whether the token was generated with the current `SECRET_KEY`.

!!! warning "User not found for token `sub`"
    The token may decode correctly, but the referenced user no longer exists (or the `sub` claim format changed).

!!! warning "Works in API client, fails in browser"
    This is often a CORS/header issue rather than a JWT issue. Confirm the browser is actually sending `Authorization`.

!!! warning "Token generated but endpoints still reject access"
    Verify the protected dependency chain and role/permission checks used by the target endpoint.

---

## Debugging Checklist

```bash
# Check runtime health first
curl http://localhost:8080/health

# Then inspect backend logs
./scripts/scipionapi logs
```

Inspect:

- auth-related exceptions
- header parsing errors
- JWT decode/validation failures
- user lookup failures

---

## Navigation

<div style="display:flex; justify-content:space-between; align-items:center; width:100%; margin-top:2rem; gap:1rem;">
  <a href="../fastapi/" style="text-decoration:none; display:inline-block;">
    ← Previous: FastAPI App and Routers
  </a>
  <a href="../database/" style="text-decoration:none; display:inline-block; margin-left:auto;">
    Next: Database and Alembic Migrations →
  </a>
</div>
