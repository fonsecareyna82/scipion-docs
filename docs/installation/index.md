---
hide:
  - toc
---

# Installation Overview

This section provides the complete installation and upgrade documentation for **ScipionAPI** and **ScipionWeb**.

Use the pages below to choose the installation workflow that best fits your environment:

- **Provision (one-shot)** for the fastest setup
- **Manual installation** for full control and debugging
- **Upgrade guide** for updating an existing installation safely

!!! note "Recommended path for most users"
    If you are installing for the first time on a local Linux machine, follow this sequence:

    1. [Prerequisites](prerequisites/)
    2. [Download and Extract Bundles](download-bundles/)
    3. [Provision (One-Shot Installation)](provision/)

---

## What Is Covered in This Section

This installation section includes:

- System prerequisites (Conda, PostgreSQL, Redis, ports, permissions)
- Official bundle download and extraction
- One-shot installation using `provision`
- Manual installation (step-by-step)
- Upgrade workflow and rollback guidance

---

## Installation Pages

### 1. Prerequisites

Prepare the machine and verify required services before installation.

- Conda installation and verification
- PostgreSQL installation and checks
- Redis installation and checks
- Ports and filesystem permissions
- Pre-installation checklist

➡️ [Open Prerequisites](prerequisites/)

---

### 2. Download and Extract Bundles

Download the official Scipion bundles and prepare the installation directory layout.

- Official download location
- API and optional Web bundle
- Recommended directory structure
- Extraction and verification steps

➡️ [Open Download and Extract Bundles](download-bundles/)

---

### 3. Provision (One-Shot Installation)

Run a complete installation using a single command.

- Conda/bootstrap + dependencies
- Database setup and migrations
- Admin user creation
- API + Celery startup
- Optional integrated Web deployment

➡️ [Open Provision (One-Shot Installation)](provision/)

---

### 4. Manual Installation

Install ScipionAPI step by step for maximum control.

- Manual Conda environment creation
- Dependency installation
- Manual PostgreSQL setup
- `.env` configuration
- Alembic migrations
- Manual API/Celery startup

➡️ [Open Manual Installation](manual-install/)

---

### 5. Upgrade Guide

Upgrade an existing installation safely.

- Stop services
- Backup database
- Replace API bundle
- Reuse `SCIPION_HOME`
- Re-run `provision`
- Optional Web bundle upgrade
- Verification and rollback steps

➡️ [Open Upgrade Guide](upgrade/)

---

## Choose the Right Workflow

=== "First-time install (recommended)"

    Follow:

    1. [Prerequisites](prerequisites/)
    2. [Download and Extract Bundles](download-bundles/)
    3. [Provision](provision/)

=== "Development / debugging"

    Follow:

    1. [Prerequisites](prerequisites/)
    2. [Download and Extract Bundles](download-bundles/)
    3. [Manual Installation](manual-install/)

=== "Upgrade existing installation"

    Follow:

    1. [Upgrade Guide](upgrade/)

---

## Quick Notes

!!! tip "Integrated mode"
    If you want the API to serve the frontend, download the **ScipionWeb** compiled bundle and use the `--web-dist` option during `provision`.

!!! warning "Backup before upgrade"
    Always create a PostgreSQL backup before running an upgrade, especially in production or shared environments.

!!! tip "Consistency matters"
    Keep **ScipionAPI** and **ScipionWeb** versions aligned whenever possible to reduce compatibility issues.

---

## Navigation

<div style="display:flex; justify-content:space-between; align-items:center; width:100%; margin-top:2rem; gap:1rem;">
  <span></span>
  <a href="../installation/prerequisites/" style="text-decoration:none; display:inline-block; margin-left:auto;">
    Next: Prerequisites →
  </a>
</div>
