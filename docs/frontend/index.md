---
hide:
  - toc
---

# Frontend Overview

ScipionWeb is the **React-based frontend** that provides the graphical user interface for ScipionAPI.

It is built with a modern web stack and distributed as a **compiled static bundle** that can be deployed in multiple modes.

!!! note "Scope of this section"
    This section covers the **frontend application model**, **build pipeline**, **runtime API URL configuration**, and **deployment options**. Backend internals are documented in the **Backend** section.

---

## Responsibilities

The frontend is responsible for:

- User authentication (login/logout)
- Project browsing
- Protocol management
- Output visualization
- Plugin interaction
- UI rendering and routing

!!! tip "Backend boundary"
    Business logic, persistence, and task execution are handled by the backend API. The frontend acts as a client application consuming REST endpoints.

---

## Technology Stack

- **UI Framework**  
  **React** + **TypeScript**

  Component-based UI with typed application code.

- **Build Tooling**  
  **Vite**

  Fast local development server and optimized production builds.

- **API Communication**  
  **REST over HTTP**

  The frontend communicates with ScipionAPI using a configurable base URL.

- **Distribution Model**  
  **Static `dist/` bundle**

  Deployable via API-integrated mode, web servers, or CDN hosting.
{: .grid .cards}

---

## Architecture

High-level request flow:

```text
Browser → React App → REST API → Backend
```

The frontend communicates with ScipionAPI through HTTP requests only.

---

## Build and Distribution Model

ScipionWeb is typically:

- Developed in a separate repository
- Built using Vite
- Packaged as a static `dist/` bundle
- Served either by:
  - ScipionAPI (integrated mode)
  - A separate web server (separate deployment)
  - CDN/static hosting

---

## Runtime API Configuration

The frontend API base URL is controlled by:

```dotenv
VITE_API_URL
```

!!! note "Build-time + runtime behavior"
    The frontend can use `VITE_API_URL` at build time and may also consume injected runtime configuration in integrated mode.

See: [Runtime API URL Configuration](runtime-config/)

---

## Deployment Modes

ScipionWeb supports three main deployment strategies:

1. **Integrated mode** (API serves frontend)
2. **Separate deployment** (frontend and API on different hosts)
3. **CDN/static hosting** (frontend on CDN, API separate)

See: [Deployment Options](deployment/)

---

## Version Compatibility

The compiled frontend bundle should match a compatible ScipionAPI version.

Recommended practice:

- Upgrade API and Web together
- Keep versioned frontend `dist` bundles
- Validate API compatibility before production rollout

!!! warning "Common upgrade pitfall"
    A frontend bundle can load successfully while still failing at runtime if the API version is incompatible (routes, payloads, auth behavior, or config expectations).

---

## Recommended Reading Order

1. [Build and Dist Output](build/)
2. [Runtime API URL Configuration](runtime-config/)
3. [Deployment Options](deployment/)

---

## Navigation

<div style="display:flex; justify-content:space-between; align-items:center; width:100%; margin-top:2rem; gap:1rem;">
  <a href="../backend/" style="text-decoration:none; display:inline-block;">
    ← Previous: Backend Overview
  </a>
  <a href="build/" style="text-decoration:none; display:inline-block; margin-left:auto;">
    Next: Build and Dist Output →
  </a>
</div>
