---
hide:
  - toc
---

# Prerequisites

Before installing **ScipionAPI** (and optionally the compiled **ScipionWeb** bundle), prepare the target machine with the required system software, services, and permissions.

This guide assumes a **Linux host** (examples use **Ubuntu/Debian**). Linux is the recommended environment for running Scipion.

!!! note "Scope"
    This page covers **system prerequisites only**. It does **not** install ScipionAPI itself yet.

---

## Overview

A typical ScipionWeb deployment requires:

- **Conda** (Miniconda or Anaconda)
- **PostgreSQL**
- **Redis**
- **sudo privileges** (recommended for automatic database bootstrap)
- **Internet access** (for Python package installation)

The installation workflow typically performs the following steps:

- Create a Conda environment
- Install Python dependencies
- Configure PostgreSQL
- Run Alembic migrations
- Start API and Celery services

---

## Supported Operating Systems

### Recommended

- Ubuntu 22.04 / 24.04
- Debian 12+

!!! tip "Other Linux distributions"
    Other Linux distributions may also work, but package names, service names, and service-management commands may differ.

---

## 1. Install Conda (Required)

ScipionAPI uses **Conda** to manage its Python environment automatically.

### Check whether Conda is already installed

```bash
conda --version
```

If this prints a version number, continue to [Install PostgreSQL](#2-install-postgresql-required).

### Install Miniconda (Recommended)

=== "Ubuntu / Debian"

    ```bash
    mkdir -p ~/Downloads
    cd ~/Downloads

    wget https://repo.anaconda.com/miniconda/Miniconda3-latest-Linux-x86_64.sh
    bash Miniconda3-latest-Linux-x86_64.sh
    ```

Follow the installer prompts.

After installation, restart your terminal or run:

```bash
conda init bash
exec bash
```

Verify the installation:

```bash
conda --version
```

### If `conda` is still not found

Add Miniconda to your `PATH` manually:

```bash
export PATH="$HOME/miniconda3/bin:$PATH"
conda --version
```

If you installed **Anaconda** instead:

```bash
export PATH="$HOME/anaconda3/bin:$PATH"
conda --version
```

!!! warning "Make PATH changes persistent"
    If the manual `PATH` export works, add it to your shell profile (for example `~/.bashrc`) so it is available in future sessions.

---

## 2. Install PostgreSQL (Required)

PostgreSQL stores application data such as:

- Users
- Projects
- Protocol metadata
- Sharing information
- Settings

### Install PostgreSQL (Ubuntu/Debian)

=== "Ubuntu / Debian"

    ```bash
    sudo apt update
    sudo apt install -y postgresql postgresql-contrib
    ```

### Start and enable PostgreSQL

```bash
sudo systemctl enable postgresql
sudo systemctl start postgresql
```

### Verify PostgreSQL is running

```bash
sudo systemctl status postgresql
```

Test the connection:

```bash
sudo -u postgres psql -c "SELECT version();"
```

### Why `sudo` matters

The installer can automatically create the PostgreSQL user and database using commands such as:

```bash
sudo -u postgres psql ...
```

This requires:

- A local PostgreSQL server (`POSTGRES_HOST=localhost`)
- `sudo` privileges

!!! note "Manual database bootstrap"
    If you prefer not to use automatic bootstrap, you can create the database and user manually before running the installer.

---

## 3. Install Redis (Required)

Redis is used as the **Celery broker** and **result backend**.

### Install Redis

=== "Ubuntu / Debian"

    ```bash
    sudo apt update
    sudo apt install -y redis-server
    ```

### Start and enable Redis

```bash
sudo systemctl enable redis-server
sudo systemctl start redis-server
```

### Verify Redis

```bash
redis-cli ping
```

Expected output:

```text
PONG
```

---

## 4. Recommended System Utilities

Make sure the following utilities are available:

```bash
sudo apt update
sudo apt install -y \
  bash \
  coreutils \
  curl \
  wget \
  unzip \
  tar \
  git \
  ca-certificates
```

!!! tip "Why these utilities?"
    They are commonly used during installation, troubleshooting, archive extraction, downloads, and repository operations.

---

## 5. Default Ports

By default, the services use:

- **API (uvicorn):** `8080`
- **PostgreSQL:** `5432`
- **Redis:** `6379`

Check whether these ports are already in use:

```bash
ss -ltnp | grep -E ':(8080|5432|6379)\b'
```

!!! warning "Port conflicts"
    If any of these ports are already in use, stop the conflicting service or adjust your ScipionAPI configuration before installation.

---

## 6. Filesystem Permissions

The installation creates a runtime workspace (`SCIPION_HOME`) that typically contains:

- `.env`
- `logs/`
- `projects/`
- `config/`
- `web/` (if using integrated web mode)

Ensure the installation user has **write permissions** to:

- The extracted **ScipionAPI** directory
- The selected `SCIPION_HOME` location

!!! caution "Avoid permission mismatches"
    Do not mix `root`-owned files with a regular-user installation directory unless you intentionally manage ownership and permissions.

---

## 7. Pre-Installation Checklist

Before continuing, confirm that:

- `conda --version` works
- PostgreSQL is running
- Redis is running
- You have `sudo` access
- Required ports are available
- Sufficient disk space is available

---

## 8. Quick Verification

Run all checks together:

```bash
conda --version
sudo systemctl is-active postgresql
sudo systemctl is-active redis-server
redis-cli ping
```

### Expected results

- A Conda version is displayed
- `active` for PostgreSQL
- `active` for Redis
- `PONG` from Redis

---

## Troubleshooting

### Conda not found

```bash
export PATH="$HOME/miniconda3/bin:$PATH"
conda init bash
exec bash
```

If the command works after exporting `PATH`, add the export line to `~/.bashrc` (or your shell profile).

### PostgreSQL not starting

```bash
sudo systemctl status postgresql
journalctl -u postgresql --no-pager -n 100
```

### Redis not responding

```bash
sudo systemctl status redis-server
journalctl -u redis-server --no-pager -n 100
```

### `sudo -u postgres` fails

Try refreshing your `sudo` session:

```bash
sudo -v
```

If it still fails, manually create the PostgreSQL user and database before running the installer.

!!! note "Common causes"
    - Your user is not in the `sudoers` group
    - `sudo` credentials expired
    - PostgreSQL service is not running
    - Local PostgreSQL authentication rules were customized

---

<div style="display:flex; justify-content:space-between; align-items:center; width:100%; margin-top:2rem; gap:1rem;">
<a href="../" style="text-decoration:none; display:inline-block;">
    ← Previous: Overview
  </a>
   <a href="../download-bundles/" style="text-decoration:none; display:inline-block; margin-left:auto;">
    Next: Download bundles (API+ Web) →
  </a>
</div>
