---
hide:
  - toc
---

# Installation Overview

Use this section to install **ScipionWeb for users**, understand the underlying packaged deployment, or upgrade an existing installation.

For a new Linux installation, the recommended path is now the **guided `install.sh` installer**. It resolves a matched API + Web release from the public release manifest, downloads the required ZIP files, verifies checksums, and runs the normal ScipionAPI provisioning workflow for you.

!!! success "Recommended path for most new installations"
    Follow this sequence:

    1. [Prerequisites](prerequisites.md)
    2. [Guided Installation](guided-install.md)

    The normal user-facing installation no longer requires manually downloading and pairing the API/Web ZIP files.

---

## Fast path

```bash
wget https://scipion.cnb.csic.es/downloads/scipion/scipionWeb/install.sh
chmod +x install.sh
./install.sh --check-only
./install.sh
```

The installer defaults to the release identified as `latest` in the public `manifest.json`.

---

## Which path should I use?

### Guided installation — recommended

Use [Guided Installation](guided-install.md) when you want:

- a normal integrated API + Web installation
- matched API and Web release versions automatically
- published SHA256 verification
- automatic port selection when no fixed port is requested
- a stable installation directory
- a guided installation marker that enables protected full uninstall later

This is the standard path for new user-facing installations.

### Manual bundle + provision workflow

Use [Download Bundles](download-bundles.md) and [Recommended Installation / Provision](provision.md) when you intentionally want to control the downloaded artifacts and run `provision` yourself.

This remains useful for:

- deployment debugging
- custom infrastructure automation
- API-only scenarios
- frontend hosted separately
- advanced testing

### Manual installation

Use [Manual Installation](manual-install.md) only when you need fine-grained control over each layer, for example:

- remote PostgreSQL setup
- unusual database/bootstrap requirements
- development environments
- custom deployment topologies
- troubleshooting low-level installation problems

### Upgrade an existing installation

Use [Upgrade / Reinstall Notes](upgrade.md) when ScipionWeb is already installed.

The normal updater preserves runtime data and configuration while updating managed API code, the deployed Web bundle, and the managed installer file.

---

## What this section covers

The installation documentation includes:

- system prerequisites such as Conda, PostgreSQL, Valkey, ports, and permissions
- guided installation through the public `install.sh`
- manual download and extraction of release bundles
- one-shot `provision` for controlled/manual installs
- low-level manual installation for advanced setups
- upgrade workflow and rollback guidance
- production deployment considerations

---

## Installation pages

### 1. Prerequisites

Prepare the machine and validate required services.

➡️ [Open Prerequisites](prerequisites.md)

### 2. Guided Installation

Install the matching API + Web release through the public installer.

➡️ [Open Guided Installation](guided-install.md)

### 3. Download and Extract Bundles

Use this when you intentionally want to manage the release ZIP files yourself.

➡️ [Open Download and Extract Bundles](download-bundles.md)

### 4. Provision

Run the complete ScipionAPI provisioning flow directly from an extracted API bundle.

➡️ [Open Provision](provision.md)

### 5. Manual Installation

Install step by step for maximum control.

➡️ [Open Manual Installation](manual-install.md)

### 6. Upgrade / Reinstall Notes

Update an existing installation safely.

➡️ [Open Upgrade / Reinstall Notes](upgrade.md)

### 7. Production Deployment

Review service-management and production deployment guidance.

➡️ [Open Production Deployment](deployment-systemd.md)

---

## Installation mental model

A useful way to think about installation is:

1. **machine readiness** → prerequisites
2. **release resolution** → `manifest.json`
3. **artifacts** → matched ScipionAPI + ScipionWeb ZIP files
4. **runtime setup** → `provision`
5. **verification** → `status`, `doctor`, browser, logs

The guided installer automates steps 2–4 while keeping the same underlying deployment model available to advanced users.

---

## Important behavior

!!! tip "API/Web versions stay paired"
    The guided installer resolves both artifacts from the same release entry. For manual installs, keep ScipionAPI and ScipionWeb versions aligned unless you are intentionally testing compatibility.

!!! tip "The API port is not assumed to be 8080"
    When no fixed API/Web port is provided, provisioning preserves an existing configured port or selects a free port automatically and persists it in `.env`.

!!! warning "Existing installation detected"
    Do not run the new-install path over an existing installation. Use `./scripts/scipionapi update` instead.

!!! warning "Backup before important upgrades"
    The updater protects application files, but database rollback still requires a database backup when schema migrations matter.

---

## Next step

Start with [Prerequisites](prerequisites.md), then use the [Guided Installation](guided-install.md) unless you specifically need a manual deployment path.
