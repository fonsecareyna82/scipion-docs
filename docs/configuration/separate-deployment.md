---
hide:
  - toc
---

# Separate Deployment (API and Web on Different Hosts)

In a separate deployment, the **API backend** and the **Web frontend** are deployed independently.

Typical architecture:

- **API** runs on one server (or service)
- **Web frontend** runs on another host, static server, or CDN

!!! note "Production-friendly pattern"
    This deployment model is common in production environments because it enables independent scaling, frontend hosting optimization, and clearer separation of responsibilities.

---

## Architecture Overview

A simplified request flow looks like this:

```text
User Browser → Web Server / CDN → REST API → PostgreSQL / Redis
```

In practice:

- The browser loads the frontend from a web host (or CDN)
- The frontend sends API requests to the ScipionAPI host
- The API talks to PostgreSQL and Redis

---

## When to Use Separate Deployment

This mode is recommended for:

- Production environments
- Cloud deployments
- Enterprise setups
- Load-balanced infrastructure
- CDN/static hosting for the frontend
- Teams that deploy frontend and backend independently

!!! tip "Best for scale and flexibility"
    If you need independent release cycles or web performance optimization, separate deployment is usually the better choice.

---

## Frontend Configuration

The frontend must know the public base URL of the API.

### Frontend `.env` (example)

```dotenv
VITE_API_URL="https://api.yourdomain.com"
```

### Frontend config usage example

```ts
export const BASE_URL = import.meta.env.VITE_API_URL;
```

After changing frontend environment variables, rebuild (or redeploy) the frontend.

!!! warning "Rebuild required"
    For most Vite-based frontends, environment variables are injected at build time. Changing `VITE_API_URL` usually requires a rebuild.

---

## API Configuration

In separate deployment mode, the API should **not** serve the frontend.

### Disable integrated mode

```dotenv
SERVE_WEB=0
```

### CORS configuration

The API must allow the frontend origin(s). Example backend configuration:

```python
allow_origins=["http://web-host:5173"]
```

Production example:

```python
allow_origins=["https://yourdomain.com"]
```

!!! warning "CORS is required"
    If CORS is not configured correctly, browser requests may fail even when the API is healthy and reachable.

---

## Production Example

### API server

```text
https://api.yourdomain.com
```

### Web server (frontend)

```text
https://yourdomain.com
```

### Example API request from the frontend

```text
https://api.yourdomain.com/api/projects
```

!!! tip "Consistent public URLs"
    Keep the frontend's configured API URL aligned with the actual public API endpoint exposed through your reverse proxy / load balancer.

---

## Networking and Security Considerations

### TLS / HTTPS

Use HTTPS on both sides in production:

- Frontend host (web app)
- API host (backend)

### Firewalls / security groups

Allow inbound access only to what is needed:

- Public web ports (`80` / `443`) on frontend host
- Public or controlled access to API host (usually `443` behind a reverse proxy)
- Internal database/Redis access restricted to trusted hosts/services

### Reverse proxy alignment

If the API is mounted under `/api`, ensure your reverse proxy configuration preserves the expected path.

!!! warning "Path rewrite mistakes"
    Misconfigured reverse proxies can strip or duplicate `/api`, causing 404s even though the backend is running.

---

## Advantages

- Independent scaling of frontend and backend
- CDN support and static hosting optimization
- Better frontend performance at scale
- Clear separation of concerns
- Independent deployment pipelines

---

## Considerations and Tradeoffs

- CORS configuration is required
- SSL/TLS must be configured correctly
- More moving parts than integrated mode
- DNS, proxy, and firewall configuration become part of deployment operations

!!! note "Operational complexity vs flexibility"
    Separate deployment adds infrastructure complexity, but it pays off in flexibility and scalability for larger environments.

---

## Recommended Configuration Checklist

### Frontend

- [ ] `VITE_API_URL` points to the correct public API URL
- [ ] Frontend rebuilt/redeployed after env changes
- [ ] HTTPS enabled on the frontend host

### API

- [ ] `SERVE_WEB=0`
- [ ] CORS allows the frontend origin(s)
- [ ] API mounted and proxied correctly (for example `/api`)
- [ ] HTTPS enabled on the public API endpoint

### Infrastructure

- [ ] DNS resolves frontend and API domains correctly
- [ ] Firewall/security groups allow required traffic only
- [ ] Reverse proxy forwards API paths as expected

---

## Verification Steps

### 1. Check API health directly

```bash
curl https://api.yourdomain.com/health
```

*(adjust the URL/path to your actual public API endpoint)*

### 2. Test a browser session

- Open the frontend URL
- Log in (if applicable)
- Open browser developer tools
- Confirm API requests go to the expected API domain/path

### 3. Validate CORS behavior

Look for browser console/network errors such as blocked cross-origin requests.

---

## Common Issues

!!! warning "CORS error in browser"
    The frontend origin is missing from the backend CORS allow list (or the scheme/port does not match exactly).

!!! warning "Frontend still calls old API host"
    `VITE_API_URL` was changed but the frontend was not rebuilt/redeployed.

!!! warning "API reachable from curl but not from browser"
    This usually indicates a CORS issue, mixed-content issue (HTTPS page calling HTTP API), or an incorrect frontend API base URL.

!!! warning "404 on `/api/...` behind reverse proxy"
    Check proxy path forwarding and backend mount path alignment.

---

## Integrated vs Separate Deployment (Quick Comparison)

=== "Integrated Mode"

    - Simpler setup
    - Single host / single endpoint
    - No CORS in common deployments
    - Easier for local/internal use

=== "Separate Deployment"

    - More flexible and scalable
    - Frontend and API deploy independently
    - CORS and proxy configuration required
    - Better fit for cloud/production platforms

---


<div style="display:flex; justify-content:space-between; align-items:center; width:100%; margin-top:2rem; gap:1rem;">
  <a href="../integrated-mode/" style="text-decoration:none; display:inline-block;">
    ← Previous: API + Web Integrated Mode
  </a>
  <span></span>
</div>
