---
hide:
  - toc
---

# Build and Dist Output

ScipionWeb frontend builds are generated with **Vite**.

This page covers:

- Local development mode
- Production builds
- `dist/` output layout
- Packaging for distribution
- Integration with ScipionAPI provisioning

---

## Development Mode

From inside the frontend repository:

```bash
npm install
npm run dev
```

This starts a local development server (typically):

```text
http://localhost:5173
```

!!! tip "Frontend-backend pairing"
    During local development, ensure the backend API is running and reachable from the browser.

---

## Development API URL

The frontend API URL should be defined in:

```text
.env
```

Example:

```dotenv
VITE_API_URL="http://localhost:8080"
```

!!! warning "Dev env changes require restart"
    If you change `.env`, restart the Vite dev server to ensure the new values are applied.

---

## Production Build

Generate the production bundle:

```bash
npm run build
```

This creates a static output directory:

```text
dist/
├── index.html
├── assets/
└── ...
```

---

## Dist Output Structure

Example production output:

```text
dist/
├── index.html
├── assets/
│   ├── index-abc123.js
│   ├── index-abc123.css
│   └── ...
└── favicon.ico
```

The `dist/` folder is static and can be:

- Served by nginx
- Served by Apache
- Hosted on a CDN/static platform
- Mounted into ScipionAPI (integrated mode)

!!! note "Hashed assets"
    File names inside `assets/` are often content-hashed for cache busting. This is expected behavior in production builds.

---

## Packaging for Distribution

To distribute the compiled frontend bundle:

```bash
zip -r ScipionWeb-<version>-dist.zip dist/
```

This ZIP can be used with ScipionAPI provisioning:

```bash
./scripts/scipionapi provision --web-dist ScipionWeb-<version>-dist.zip
```

!!! tip "Versioned artifacts"
    Keep versioned ZIP files (and optionally build metadata) to simplify rollback and compatibility tracking.

---

## Build Verification Checklist

After `npm run build`, verify:

- `dist/index.html` exists
- `dist/assets/` contains JS/CSS bundles
- The configured API URL strategy matches your deployment mode
- The bundle version matches the target API version

---

## Important Notes

!!! warning "Do not edit `dist/` manually"
    Treat `dist/` as generated output. Make source changes in the frontend codebase and rebuild.

!!! warning "Rebuild after config changes"
    Changes to `VITE_API_URL` and other build-time env values require a new production build.

!!! warning "Check API compatibility"
    Ensure the frontend bundle and ScipionAPI version are compatible before production deployment.

---

## Navigation

<div style="display:flex; justify-content:space-between; align-items:center; width:100%; margin-top:2rem; gap:1rem;">
  <a href="../" style="text-decoration:none; display:inline-block;">
    ← Previous: Frontend Overview
  </a>
  <a href="../runtime-config/" style="text-decoration:none; display:inline-block; margin-left:auto;">
    Next: Runtime API URL Configuration →
  </a>
</div>
