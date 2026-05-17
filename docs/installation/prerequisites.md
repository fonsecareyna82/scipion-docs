# Prerequisites

Before installing **ScipionAPI** and optionally the compiled **ScipionWeb** bundle, prepare the target machine with the required system software, services, and permissions.

This guide assumes a **Linux host**. Ubuntu and Debian are the most straightforward environments for the documented commands.

!!! note "Scope"
    This page covers **system prerequisites only**. It does not install ScipionAPI itself yet.

---

## What you need before installation

A typical ScipionWeb deployment requires:

- **Conda** (Miniconda or Anaconda)
- **PostgreSQL**
- **Redis**
- **sudo privileges** in common local-install scenarios
- **Internet access** for Python package installation

The installation flow later uses these pieces to:

- create the Python environment
- configure the database
- run migrations
- start API and Celery services

---

## Supported Operating Systems

### Recommended

- Ubuntu 22.04 / 24.04
- Debian 12+

!!! tip "Other Linux distributions"
    Other Linux distributions may also work, but package names, service names, and service-management commands may differ.

---

## 1. Install Conda

ScipionAPI uses **Conda** to manage its Python environment automatically.

### Check whether Conda is already installed

```bash
conda --version
```

If that prints a version number, continue to PostgreSQL.

### Install Miniconda (recommended)

If `conda` is not available, install Miniconda:

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

Then verify again:

```bash
conda --version
```

### If `conda` is still not found

Add Miniconda to your `PATH` manually:

```bash
export PATH="$HOME/miniconda3/bin:$PATH"
conda --version
```

If you installed **Anaconda** instead, adjust the path accordingly:

```bash
export PATH="$HOME/anaconda3/bin:$PATH"
conda --version
```

!!! warning "Make PATH changes persistent"
    If `conda` only works after manually editing `PATH`, add that change to your shell profile before continuing.

---

## 2. Install PostgreSQL

PostgreSQL stores application data such as:

- users
- projects
- protocol metadata
- sharing information
- settings

In common local setups, the installer can create the database and role automatically, but only if PostgreSQL is installed, running, and reachable.

### Install PostgreSQL on Ubuntu or Debian

```bash
sudo apt update
sudo apt install -y postgresql postgresql-contrib
```

### Start and enable PostgreSQL

```bash
sudo systemctl enable postgresql
sudo systemctl start postgresql
```

### Verify PostgreSQL

```bash
sudo systemctl status postgresql
sudo -u postgres psql -c "SELECT version();"
```

### Why sudo matters

The installer can automatically create the PostgreSQL user and database in common local setups using commands such as:

```bash
sudo -u postgres psql ...
```

This usually requires:

- a local PostgreSQL server
- a valid sudo session
- permission to run commands as the `postgres` system user

!!! note "Manual database bootstrap"
    If you prefer not to use automatic database bootstrap, create the database and user manually before running the installer.

---

## 3. Install Redis

Redis is used as the **Celery broker** and **result backend**.

### Install Redis on Ubuntu or Debian

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
sudo systemctl status redis-server
redis-cli ping
```

Expected output:

```text
PONG
```

---

## 4. Install recommended system utilities

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

## 5. Verify ports and permissions

By default, the services use:

- **API:** `8080`
- **PostgreSQL:** `5432`
- **Redis:** `6379`

Check whether those ports are already in use:

```bash
ss -ltnp | grep -E ':(8080|5432|6379)\b'
```

Also confirm that the installation user will have write access to:

- the extracted ScipionAPI directory
- the selected `SCIPION_HOME` location

!!! caution "Avoid permission mismatches"
    Do not mix `root`-owned files with a regular-user installation directory unless you intentionally manage ownership and permissions.

---

## 6. Pre-installation checklist

Before continuing, confirm that:

- `conda --version` works
- PostgreSQL is installed and running
- Redis is installed and running
- required utilities such as `curl`, `wget`, and `unzip` are available
- you have `sudo` access if you want automatic local DB bootstrap
- required ports are available
- disk space is sufficient for bundles, runtime data, and logs

---

## Fast sanity check

Run these together:

```bash
conda --version
sudo systemctl is-active postgresql
sudo systemctl is-active redis-server
redis-cli ping
```

You want to see:

- a Conda version
- `active` for PostgreSQL
- `active` for Redis
- `PONG` from Redis

---

## Common prerequisite failures

### Conda not found

Usually a PATH or shell-initialization issue.

```bash
export PATH="$HOME/miniconda3/bin:$PATH"
conda init bash
exec bash
```

If the command works after exporting `PATH`, add the export line to your shell profile.

### PostgreSQL not starting

Inspect the service status and recent logs:

```bash
sudo systemctl status postgresql
journalctl -u postgresql --no-pager -n 100
```

### Redis not responding

Inspect the service status and recent logs:

```bash
sudo systemctl status redis-server
journalctl -u redis-server --no-pager -n 100
```

### `sudo -u postgres` fails

Try refreshing your sudo session:

```bash
sudo -v
```

If it still fails, manually create the PostgreSQL user and database before running the installer.

Common causes include:

- your user is not in the `sudoers` group
- sudo credentials expired
- PostgreSQL service is not running
- local PostgreSQL authentication rules were customized
