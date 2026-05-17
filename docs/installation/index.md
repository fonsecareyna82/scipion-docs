---
hide:
  - toc
---

# Installation Overview

Use this section when you want to install **ScipionWeb for users** or upgrade an existing installation.

For most installations, the recommended path is **Integrated Mode (API + Web)**. In that mode, ScipionAPI serves the compiled ScipionWeb interface and the API from the same runtime, so users can open the web application directly in the browser.

!!! note "Recommended path for most installations"
    If you are installing for the first time on a local Linux machine, follow this sequence:

    1. [Prerequisites](prerequisites/)
    2. [Download and Extract Bundles](download-bundles/)
    3. [Recommended Installation](provision/)

---

## Which path should I use?

### Recommended installation

Use **Prerequisites → Download Bundles → Recommended Installation** when you want the fastest path to a working ScipionWeb instance with both API and Web UI.

This is the normal path for user-facing installations.

### Manual installation

Use **Manual Installation** only when you need full control over each layer, for example:

- remote PostgreSQL setup
- debugging installation problems
- custom deployment requirements
- development environments

### Upgrade or reinstall

Use **Upgrade / Reinstall Notes** when an existing installation already exists and you need to preserve runtime data, database state, and configuration.

---

## What this section covers

This installation section includes:

- system prerequisites such as Conda, PostgreSQL, Redis, ports, and permissions
- official bundle download and extraction
- recommended integrated installation using `provision` and `--web-dist`
- manual installation step by step for advanced setups
- upgrade workflow and rollback guidance

---

## Installation pages

### 1. Prerequisites

Prepare the machine and verify required services before installation.

➡️ [Open Prerequisites](prerequisites/)

### 2. Download and Extract Bundles

Download the official ScipionAPI and ScipionWeb bundles and prepare the installation directory layout.

➡️ [Open Download and Extract Bundles](download-bundles/)

### 3. Recommended Installation

Run a complete integrated installation using a single command.

➡️ [Open Recommended Installation](provision/)

### 4. Manual Installation

Install ScipionAPI step by step for maximum control.

➡️ [Open Manual Installation](manual-install/)

### 5. Upgrade / Reinstall Notes

Upgrade an existing installation safely.

➡️ [Open Upgrade / Reinstall Notes](upgrade/)

---

## Common installation split

A useful way to think about installation is:

1. **machine readiness** → prerequisites
2. **artifacts** → bundles
3. **runtime setup** → recommended installation or manual installation
4. **service verification** → browser, health endpoint, logs, and project loading

This mental split makes debugging much easier when something goes wrong.

---

## Quick notes

!!! tip "Integrated mode is the normal user-facing mode"
    If you want users to open ScipionWeb in the browser, download the **ScipionWeb** compiled bundle and use the `--web-dist` option during `provision`.

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
