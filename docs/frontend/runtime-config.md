---
hide:
  - toc
---

# Runtime API URL Configuration

The frontend communicates with ScipionAPI using a base API URL.

This is primarily defined by:

```dotenv
VITE_API_URL
```

---

## Build-Time Configuration

### Development

```dotenv
VITE_API_URL="http://localhost:8080"
```

### Production (separate deployment example)

```dotenv
VITE_API_URL="https://api.yourdomain.com"
```

The variable is typically consumed as:

```ts
export const BASE_URL = import.meta.env.VITE_API_URL;
```

!!! note "Vite behavior"
    Values from `VITE_*` variables are injected into the frontend build and are not automatically updated after build time.

---

## Integrated Mode (API + Web)

In integrated mode:

- Web is served at `/`
- API is mounted at `/api`

No external API host is required for the frontend.

The installer can inject runtime configuration through:

```text
window.__SCIPION_WEB_CONFIG__
```

Example payload:

```json
{
  "apiBaseUrl": "/api"
}
```

The frontend can prefer this runtime value when present, allowing a single build artifact to adapt to integrated deployments.

!!! tip "Why runtime injection helps"
    Runtime config avoids rebuilding the frontend just to change the API mount path in integrated scenarios.

---

## Separate Deployment

If frontend and API run on different hosts, define the API URL in the frontend `.env` before building:

```dotenv
VITE_API_URL="https://api.yourdomain.com"
```

Then rebuild the frontend:

```bash
npm run build
```

---

## Configuration Matrix

| Deployment Mode | Frontend Host | API Host | Typical API Base |
|---|---|---|---|
| Integrated | Same as API | Same as frontend | `/api` |
| Separate | `https://yourdomain.com` | `https://api.yourdomain.com` | `https://api.yourdomain.com` |
| CDN | CDN/static host | Separate API host | `https://api.yourdomain.com` |

---

## Common Mistakes

!!! warning "Changing `.env` without rebuilding"
    In production builds, `VITE_API_URL` is compiled into the bundle. Rebuild after changes.

!!! warning "HTTP/HTTPS mismatch"
    Mixed protocol usage (frontend on HTTPS, API on HTTP) can break requests in browsers.

!!! warning "CORS not configured on backend"
    Separate deployments require backend CORS configuration for the frontend origin.

!!! warning "Using localhost in production builds"
    Double-check build artifacts before release to avoid accidentally shipping `localhost` API URLs.

---

## Production Recommendations

- Use HTTPS for frontend and API
- Use stable domain naming (`app.` / `api.` pattern, or integrated `/api`)
- Document your API URL strategy per environment (dev/staging/prod)
- Validate API reachability from the browser before rollout

---

## Quick Validation

After deployment, confirm in the browser:

- Login requests go to the expected API origin/path
- No CORS errors appear in DevTools
- No mixed-content warnings appear
- Network requests return expected responses

---

## Navigation

<div style="display:flex; justify-content:space-between; align-items:center; width:100%; margin-top:2rem; gap:1rem;">
  <a href="../build/" style="text-decoration:none; display:inline-block;">
    ← Previous: Build and Dist Output
  </a>
  <a href="../deployment/" style="text-decoration:none; display:inline-block; margin-left:auto;">
    Next: Deployment Options →
  </a>
</div>
