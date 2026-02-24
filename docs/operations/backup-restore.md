---
hide:
  - toc
---

# Backup and Restore

ScipionWeb disaster recovery requires backing up **both** the database and the runtime workspace.

A complete backup includes:

1. **PostgreSQL dump** (metadata, users, settings, tags, sharing, protocol records)
2. **`SCIPION_HOME` backup** (projects, logs, runtime config, deployed web bundle, `.env`)

!!! warning "Full recovery requires both parts"
    Backing up only PostgreSQL or only `SCIPION_HOME` is **not sufficient** for a reliable restore.

---

## What Must Be Backed Up

### PostgreSQL Database

The database stores application metadata such as:

- Users
- Project metadata
- Protocol metadata
- Sharing information
- Settings
- Tags

### `SCIPION_HOME`

The runtime workspace stores operational data such as:

- `projects/`
- `logs/`
- `config/`
- `web/`
- `.env`

!!! tip "Keep backups separate"
    Store database dumps and filesystem archives as separate artifacts. This makes restores and validation easier.

---

## Backup Strategy Overview

Recommended baseline strategy:

- **Daily** PostgreSQL backups
- **Weekly** `SCIPION_HOME` filesystem backups
- **Off-site / external storage**
- **Regular restore testing** (do not skip this)

| Backup Component | Frequency | Why It Matters |
|---|---:|---|
| PostgreSQL dump | Daily | Protects users, metadata, settings, tags |
| `SCIPION_HOME` archive | Weekly (or daily in active systems) | Protects projects, logs, runtime config, `.env` |
| Restore verification | Scheduled | Confirms backups are actually usable |

---

## Backup the Database

### Plain SQL dump

```bash
pg_dump -U scipion_user scipion_db > scipion_backup.sql
```

### Compressed dump

```bash
pg_dump -U scipion_user scipion_db | gzip > scipion_backup.sql.gz
```

!!! note "Compressed backups"
    Compressed dumps are usually better for storage efficiency and transfer.

---

## Backup `SCIPION_HOME`

Create a compressed archive:

```bash
tar -czf scipion_home_backup.tar.gz scipion_home/
```

If your `SCIPION_HOME` lives elsewhere, use the absolute path:

```bash
tar -czf scipion_home_backup.tar.gz /opt/scipionweb/scipion_home/
```

---

## Full Backup Script Example

Minimal one-shot example:

```bash
pg_dump -U scipion_user scipion_db | gzip > db_backup.sql.gz
tar -czf scipion_home_backup.tar.gz scipion_home/
```

!!! warning "Store backups outside the host"
    Keep copies in a secure external location (separate disk, backup server, object storage, etc.). Backups on the same machine are not enough for disaster recovery.

---

## Restore Workflow Overview

Recommended restore order:

1. Restore **database**
2. Restore **`SCIPION_HOME`**
3. Validate `.env` and permissions
4. Start services
5. Verify `/health` and logs

!!! tip "Restore to staging first"
    When possible, test recovery in an isolated environment before touching production.

---

## Restore the Database

### Step 1: Create an empty database

```bash
createdb -U scipion_user scipion_db
```

### Step 2: Restore from plain SQL dump

```bash
psql -U scipion_user scipion_db < scipion_backup.sql
```

### Step 3: Restore from compressed dump (if applicable)

```bash
gunzip -c db_backup.sql.gz | psql -U scipion_user scipion_db
```

!!! note "Existing database"
    If the target database already exists and contains data, ensure you are restoring into the correct target and expected state before executing restore commands.

---

## Restore `SCIPION_HOME`

Extract the archive:

```bash
tar -xzf scipion_home_backup.tar.gz
```

Ensure the service user can read/write the restored workspace:

```bash
chmod -R u+rw scipion_home
```

If using a dedicated service account, also fix ownership as needed:

```bash
chown -R youruser:yourgroup scipion_home
```

---

## Post-Restore Validation

After restoring both components:

1. Verify `.env` exists and matches the intended deployment
2. Confirm `SECRET_KEY` consistency
3. Start services
4. Check health endpoint
5. Inspect logs

### Health check

```bash
curl http://localhost:8080/health
```

Expected:

```json
{"status":"ok"}
```

### Runtime checks

```bash
./scripts/scipionapi status
./scripts/scipionapi logs
```

---

## Disaster Recovery Checklist

Use this checklist during incidents:

- [ ] Restore PostgreSQL dump
- [ ] Restore `SCIPION_HOME`
- [ ] Validate `.env`
- [ ] Confirm `SECRET_KEY` matches expected deployment
- [ ] Fix permissions/ownership
- [ ] Start services
- [ ] Verify `/health`
- [ ] Verify frontend access
- [ ] Inspect `app.log` / `celery.log`

---

## Production Notes

!!! warning "Backups without verification are not backups"
    Schedule regular restore drills (staging is enough) to validate credentials, file permissions, and actual recovery time.

!!! tip "Version discipline"
    Keep backup timestamps and version labels (API version, frontend bundle version, schema revision if relevant) to simplify rollback and incident response.

---

## Navigation

<div style="display:flex; justify-content:space-between; align-items:center; width:100%; margin-top:2rem; gap:1rem;">
  <span></span>
  <a href="../logs-and-pids/" style="text-decoration:none; display:inline-block; margin-left:auto;">
    Next: Logs and PID Files →
  </a>
</div>
