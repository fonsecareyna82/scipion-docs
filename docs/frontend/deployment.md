---
hide:
  - toc
---

# Deployment Options

ScipionWeb frontend can be deployed in multiple ways depending on scale, operational needs, and infrastructure constraints.

This page covers:

1. Integrated mode
2. Separate deployment
3. CDN/static hosting
4. Reverse proxy example (nginx)
5. Production checklist and recommendations

---

## Option 1: Integrated Mode

Frontend is served by the API.

Example provisioning command:

```bash
./scripts/scipionapi provision \
  --user admin \
  --email admin@example.com \
  --pass changeMe \
  --web-dist ScipionWeb-<version>-dist.zip
```

Access:

```text
http://localhost:8080/
```

### Advantages

- Single service entrypoint
- No CORS configuration (same origin)
- Simple installation and maintenance
- Good fit for local labs and small deployments

!!! tip "Recommended for simple deployments"
    Integrated mode is usually the fastest and safest path for first-time installations.

---

## Option 2: Separate Deployment

Frontend and API run independently.

Example topology:

```text
Frontend: https://yourdomain.com
API:      https://api.yourdomain.com
```

### Requirements

- Proper backend CORS configuration
- Correct frontend API URL (`VITE_API_URL`)
- HTTPS enabled (recommended/expected for production)

### Advantages

- Independent scaling
- Flexibility in infrastructure choices
- Easier CDN/static frontend hosting
- Better performance tuning at scale

---

## Option 3: CDN / Static Hosting

Frontend is hosted on a static platform (for example):

- Cloudflare
- AWS S3 (with static hosting/CDN)
- Azure Blob static hosting
- Other static hosting providers

API remains hosted separately.

### Recommended for

- Internet-facing deployments
- Multi-site / distributed users
- Larger-scale environments
- Performance-focused setups

---

## Reverse Proxy Example (nginx)

Example with frontend static files + proxied API:

```nginx
server {
    listen 80;
    server_name yourdomain.com;

    location / {
        root /var/www/scipionweb/dist;
        try_files $uri /index.html;
    }

    location /api/ {
        proxy_pass http://127.0.0.1:8080/api/;
    }
}
```

!!! note "SPA routing"
    `try_files $uri /index.html;` is important for React/Vite SPA routing so deep links resolve correctly.

---

## Deployment Decision Guide

- **Integrated mode** → simplest installs, internal/lab usage, quick rollout
- **Separate deployment** → production web/API separation, scaling flexibility
- **CDN/static hosting** → high-performance public delivery, enterprise-style setups

---

## Production Checklist

- HTTPS enabled
- Correct API base URL configured
- Backend CORS configured (separate/CDN modes)
- API reachable from frontend runtime
- No mixed HTTP/HTTPS content
- Frontend bundle version compatible with API
- Reverse proxy configured for SPA routing (if applicable)

---

## Recommendation Summary

| Deployment Type | Recommended For |
|---|---|
| Integrated | Local labs / simple installs |
| Separate | Production servers |
| CDN | Enterprise scale / public delivery |

---

## Navigation

<div style="display:flex; justify-content:space-between; align-items:center; width:100%; margin-top:2rem; gap:1rem;">
  <a href="../runtime-config/" style="text-decoration:none; display:inline-block;">
    ← Previous: Runtime API URL Configuration
  </a>
  <span></span>
</div>
