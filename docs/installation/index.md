---
hide:
  - toc
---

# Installation Overview

This section provides the complete installation and upgrade documentation for **ScipionAPI** and **ScipionWeb**.

Use the pages below to choose the workflow that best fits your environment:

- **Provision (one-shot)** for the fastest supported setup
- **Manual installation** for full control and debugging
- **Upgrade guide** for updating an existing installation safely

!!! note "Recommended path for most users"
    If you are installing for the first time on a local Linux machine, follow this sequence:

    1. [Prerequisites](prerequisites/)
    2. [Download and Extract Bundles](download-bundles/)
    3. [Provision (One-Shot Installation)](provision/)

---

## How to choose the right path

### First-time install

Use **Prerequisites → Download Bundles → Provision**.

This is the shortest path from zero to a working installation.

### Development or deep debugging

Use **Prerequisites → Download Bundles → Manual Installation**.

This path is slower, but much better when you need to inspect each layer independently.

### Existing installation upgrade

Use **Upgrade Guide**.

Upgrades should be treated as controlled operations with backup and verification.

---

## What this section covers

This installation section includes:

- system prerequisites such as Conda, PostgreSQL, Redis, ports, and permissions
- official bundle download and extraction
- one-shot installation using `provision`
- manual installation step by step
- upgrade workflow and rollback guidance

---

## Installation Pages

### 1. Prerequisites

Prepare the machine and verify required services before installation.

➡️ [Open Prerequisites](prerequisites/)

### 2. Download and Extract Bundles

Download the official Scipion bundles and prepare the installation directory layout.

➡️ [Open Download and Extract Bundles](download-bundles/)

### 3. Provision (One-Shot Installation)

Run a complete installation using a single command.

➡️ [Open Provision (One-Shot Installation)](provision/)

### 4. Manual Installation

Install ScipionAPI step by step for maximum control.

➡️ [Open Manual Installation](manual-install/)

### 5. Upgrade Guide

Upgrade an existing installation safely.

➡️ [Open Upgrade Guide](upgrade/)

---

## Common installation split

A useful way to think about installation is:

1. **machine readiness** → prerequisites
2. **artifacts** → bundles
3. **runtime setup** → provision or manual install
4. **service verification** → health, logs, and browser checks

This mental split makes debugging much easier when something goes wrong.

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
