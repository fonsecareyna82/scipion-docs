---
hide:
  - toc
---

# Security Notes

This page summarizes practical security guidance for deploying ScipionWeb in production.

!!! warning "Shared responsibility"
    Security depends on both the application **and** the deployment environment (reverse proxy, firewall, service user permissions, secrets handling, and monitoring).

---

## Security Priorities (At a Glance)

- Use **HTTPS**
- Protect `SECRET_KEY`
- Restrict **CORS**
- Keep PostgreSQL and Redis **non-public**
- Enforce proper file permissions
- Monitor logs and automate backups

---

## 1. Use HTTPS

Never expose the API over plain HTTP in production.

Recommended setup:

- Reverse proxy (for example **nginx**)
- TLS certificates (Let's Encrypt or enterprise CA)

Example public endpoints:

```text
https://yourdomain.com
https://api.yourdomain.com
```

!!! tip "Why HTTPS is mandatory"
    HTTPS protects credentials, tokens, and session traffic from interception.

---

## 2. Protect `SECRET_KEY`

`SECRET_KEY` is used to sign JWT tokens.

It:

- Must remain private
- Must not be committed to version control
- Should be rotated if compromised

Recommended practices:

- Use a strong random value (**32+ characters minimum**)
- Store it in `.env` with restricted permissions
- Consider injecting it via `systemd` environment configuration for additional hardening

!!! warning "Compromise response"
    If `SECRET_KEY` is exposed, rotate it and assume existing tokens may be untrusted.

---

## 3. Restrict CORS

Avoid permissive production CORS policies such as:

```python
allow_origins=["*"]
```

Use explicit origins instead:

```python
allow_origins=["https://yourdomain.com"]
```

If you use separate frontend/API hosts, include only the required frontend origins.

---

## 4. Database Security (PostgreSQL)

Recommended controls:

- Use strong passwords
- Bind PostgreSQL to `localhost` when possible
- Do **not** expose port `5432` publicly
- Apply firewall rules
- Limit remote access to trusted hosts only (if remote DB is required)

!!! tip "Defense in depth"
    Even if PostgreSQL requires credentials, network exposure should still be minimized.

---

## 5. Redis Security

Recommended controls:

- Bind Redis to `localhost`
- Do **not** expose Redis publicly
- Consider authentication if Redis must be remote
- Restrict network access with firewall rules

!!! warning "High-risk exposure"
    Public Redis exposure is a common operational mistake and can lead to severe compromise.

---

## 6. Firewall Configuration

Typical port policy:

- **80 / 443** → public (if internet-facing)
- **8080** → internal only (when reverse proxy is used)
- **5432** (PostgreSQL) → **not public**
- **6379** (Redis) → **not public**

Use host firewall rules and/or cloud security groups to enforce this.

---

## 7. JWT Security Practices

JWT security depends on both backend behavior and deployment choices.

Recommended:

- Use HTTPS everywhere
- Use short-lived access tokens
- Consider refresh-token strategy
- Invalidate access when users are disabled or permissions change (application-policy dependent)

---

## 8. File Permissions

Protect runtime configuration and data:

- `.env` should be readable only by the service user
- `scipion_home` should not be world-writable
- Ensure correct ownership for the runtime user/service

Example:

```bash
chmod 600 scipion_home/.env
chmod -R 750 scipion_home
```

Optional ownership fix:

```bash
chown -R youruser:yourgroup scipion_home
```

---

## 9. Production Hardening Checklist

- [ ] HTTPS enabled
- [ ] Reverse proxy configured
- [ ] `SECRET_KEY` secured
- [ ] CORS restricted to trusted origins
- [ ] PostgreSQL not publicly exposed
- [ ] Redis not publicly exposed
- [ ] Runtime permissions hardened (`.env`, `SCIPION_HOME`)
- [ ] Logs monitored
- [ ] Backups automated and tested

---

## 10. Security Model Summary

ScipionWeb security relies on:

- JWT authentication
- Backend validation
- Secure runtime configuration
- Deployment hardening (network, proxy, permissions, monitoring)

!!! note "Operational reality"
    Security is not a one-time setup. Revisit your configuration after upgrades, topology changes, or new public exposure.

---

## Navigation

<div style="display:flex; justify-content:space-between; align-items:center; width:100%; margin-top:2rem; gap:1rem;">
  <a href="../logs-and-pids/" style="text-decoration:none; display:inline-block;">
    ← Previous: Logs and PID Files
  </a>
  <span></span>
</div>
