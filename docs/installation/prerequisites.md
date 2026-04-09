---
hide:
  - toc
---

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

Check whether Conda is already installed:

```bash
conda --version
```

If that works, continue to PostgreSQL.

If not, install Miniconda, initialize your shell, and verify the command again.

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

Useful checks:

```bash
sudo systemctl status postgresql
sudo -u postgres psql -c "SELECT version();"
```

---

## 3. Install Redis

Redis is used as the **Celery broker** and **result backend**.

Useful checks:

```bash
sudo systemctl status redis-server
redis-cli ping
```

Expected output:

```text
PONG
```

---

## 4. Verify ports and permissions

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

---

## 5. Pre-installation checklist

Before continuing, confirm that:

- `conda --version` works
- PostgreSQL is running
- Redis is running
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

### PostgreSQL not starting

Usually a service or package-level problem that should be fixed before running any Scipion installer.

### Redis not responding

Background task execution will not work correctly until Redis is healthy.

### `sudo -u postgres` fails

Automatic local database bootstrap may fail if your `sudo` session is not valid or PostgreSQL auth rules were customized.
