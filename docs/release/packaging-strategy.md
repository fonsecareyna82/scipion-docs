# Packaging Strategy

This section describes how ScipionWeb is packaged and released as installable bundles.

The goal is to provide a **human-friendly installation experience**:

- No git cloning required
- No manual dependency wiring
- One-shot provisioning
- Clean upgrades

---

# High-Level Concept

Each release consists of two artifacts:

| Bundle | Purpose |
|-------|--------|
| ScipionAPI bundle | Backend runtime + installer |
| ScipionWeb bundle | Pre-built frontend (Vite dist) |

Users download, extract, provision, and run.

---

# Bundle Layout

## API Bundle

```
scipionapi-X.Y.Z/
├── alembic/
├── app/
├── scipionapi_cli/
├── scripts/
├── requirements.txt
├── pyproject.toml
└── README.md
```

## Web Bundle

```
scipionweb-X.Y.Z/
└── dist/
    ├── index.html
    ├── assets/
    └── config.js (runtime injected)
```

---

# Why Two Bundles?

Separation enables:

- Independent scaling
- Remote API deployments
- CDN web hosting
- Easier upgrades

But also supports:

- Integrated mode (API serves web)

---

# Supported Deployment Models

| Model | Description |
|------|------------|
| Integrated | API + Web on same host |
| Separate | API server + Web static host |
| Hybrid | API private + Web public |

---

# Versioning Strategy

Recommended:

```
MAJOR.MINOR.PATCH
```

Example:

```
1.2.0
```

Where:

- MAJOR: breaking schema/runtime
- MINOR: features
- PATCH: fixes

---

# Release Artifact Naming

```
scipionapi-1.2.0-linux-x86_64.zip
scipionweb-1.2.0-dist.zip
```

---

# Distribution Endpoint

Example:

```
https://scipion.cnb.csic.es/downloads/scipion/scipionWeb/
```

---

# Design Principles

✔ No git required  
✔ No manual Python env juggling  
✔ Deterministic installs  
✔ Reproducible runtime  
✔ Human-friendly commands  

---

# Summary

ScipionWeb follows a **bundle + provision** deployment philosophy:

> Download → Extract → Provision → Run