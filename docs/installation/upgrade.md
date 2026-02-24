---
hide:
  - toc
---

# Upgrade Guide

This guide explains how to upgrade **ScipionAPI** and, optionally, **ScipionWeb** to a newer version.

!!! note "Recommended approach"
    Treat upgrades as a controlled operation: **stop services**, **backup first**, **upgrade**, then **verify** before resuming normal usage.

---

## Overview

A typical upgrade involves:

1. Stopping services
2. Backing up the database
3. Replacing the API bundle
4. Reusing the existing `SCIPION_HOME`
5. Running migrations (via `provision`)
6. Restarting services
7. Optionally replacing the Web bundle

!!! tip "Why `provision` again?"
    The `provision` command is the recommended upgrade path because it can reuse the existing environment, apply dependency updates, and run Alembic migrations safely.

---

## 1. Stop Services

From the current installation directory, stop the running services:

```bash
./scripts/scipionapi stop
```

Verify they are stopped:

```bash
./scripts/scipionapi status
```

!!! warning "Do not upgrade while services are running"
    Upgrading code or schema while the API/Celery worker is still running can cause inconsistent behavior and migration conflicts.

---

## 2. Backup Database (Strongly Recommended)

Create a PostgreSQL backup before upgrading.

### Plain SQL backup

```bash
pg_dump -U scipion_user scipion_db > scipion_backup.sql
```

### Compressed backup

```bash
pg_dump -U scipion_user scipion_db | gzip > scipion_backup.sql.gz
```

Store the backup in a safe location (preferably outside the installation directory).

!!! tip "Production safety"
    If this is a production deployment, also consider taking a filesystem backup of `SCIPION_HOME` (especially `.env`, `projects/`, and custom configuration files).

---

## 3. Download the New Version

Download the new bundles from the official release location:

[https://scipion.cnb.csic.es/downloads/scipion/scipionWeb/](https://scipion.cnb.csic.es/downloads/scipion/scipionWeb/)

Extract the new **ScipionAPI** version:

```bash
unzip ScipionAPI-<new_version>.zip
cd ScipionAPI-<new_version>
```

!!! note "Web bundle is optional"
    Download the new `ScipionWeb-<new_version>-dist.zip` only if you are using **integrated mode** (API serving the frontend).

---

## 4. Reuse Existing `SCIPION_HOME`

Do **not** delete your existing `scipion_home`.

You can either:

- Copy the existing `scipion_home` into the new API directory, **or**
- Export `SCIPION_HOME` explicitly to point to the existing runtime workspace

Example:

```bash
export SCIPION_HOME=/path/to/previous/scipion_home
```

Your runtime data remains intact, including:

- `.env`
- `logs/`
- `projects/`
- deployed web assets (if previously deployed)
- local runtime configuration

!!! warning "Preserve your `.env`"
    The `.env` file contains database and runtime configuration. Verify it points to the correct database and services before running the upgrade.

---

## 5. Upgrade the Environment and Database

Run `provision` again from the **new API bundle directory**:

```bash
./scripts/scipionapi provision \
  --user "admin" \
  --email "admin@example.com" \
  --pass "changeMe"
```

This typically:

- Reuses the existing Conda environment (if already present)
- Installs updated Python dependencies
- Applies new Alembic migrations
- Keeps your database data intact
- Restarts API and Celery services

!!! tip "Admin credentials"
    `provision` may update the admin credentials if needed. Use the same admin account or set a new temporary password and rotate it later.

---

## 6. Upgrade Web Bundle (Optional, Integrated Mode)

If you are using **integrated mode**, re-run `provision` and pass the new Web bundle ZIP:

```bash
./scripts/scipionapi provision \
  --user "admin" \
  --email "admin@example.com" \
  --pass "changeMe" \
  --web-dist /path/to/ScipionWeb-<new_version>-dist.zip
```

This replaces the deployed frontend assets with the new version.

!!! tip "Keep API and Web versions aligned"
    Whenever possible, upgrade **ScipionAPI** and **ScipionWeb** together to avoid frontend/API compatibility mismatches.

---

## 7. Verify the Upgrade

### Check service status

```bash
./scripts/scipionapi status
```

### Check API health

```bash
curl http://localhost:8080/health
```

Expected response:

```json
{"status":"ok"}
```

### Check API docs

Open in your browser:

- **API-only mode:** `http://localhost:8080/docs`
- **Integrated mode:** `http://localhost:8080/api/docs`

!!! tip "Post-upgrade smoke test"
    Log in, open a project, and run a basic operation to confirm the application behaves correctly after the upgrade.

---

## Rolling Back (If Needed)

If the upgrade fails and you need to roll back:

1. Stop services
2. Restore the database backup
3. Revert to the previous API bundle
4. Start services again

### Restore a plain SQL backup

```bash
psql -U scipion_user scipion_db < scipion_backup.sql
```

### Restore a compressed backup

```bash
gunzip -c scipion_backup.sql.gz | psql -U scipion_user scipion_db
```

!!! warning "Schema mismatch risk"
    If you roll back the API code, also ensure the database is restored to a compatible pre-upgrade state. Running old code against a migrated schema may fail.

---

## Best Practices

- Always create a backup before upgrading
- Test upgrades in staging first
- Keep the previous API bundle until the upgrade is verified
- Review migration scripts before production deployment
- Verify available disk space before extracting new bundles and backups

---

## Major Version Upgrades

If upgrading across **major versions**:

- Read release notes carefully
- Verify migration compatibility
- Test in an isolated environment first
- Expect possible schema changes and behavior changes

!!! note "Change management"
    For major upgrades, plan a maintenance window and a rollback strategy before starting.

---

## Summary

Recommended upgrade flow:

1. Stop services
2. Backup database
3. Download and extract the new API bundle
4. Reuse existing `SCIPION_HOME`
5. Run `provision`
6. Upgrade the Web bundle (optional)
7. Verify services and application behavior

Upgrade is generally safe when migrations are applied in order and verified backups are available.

---

<div style="display:flex; justify-content:space-between; align-items:center; width:100%; margin-top:2rem; gap:1rem;">
  <a href="../manual-install/" style="text-decoration:none; display:inline-block;">
    ← Previous: Manual Installation
  </a>
  <a href="../deployment-systemd/" style="text-decoration:none; display:inline-block; margin-left:auto;">
    Next: Production Deployment →
  </a>
</div>
