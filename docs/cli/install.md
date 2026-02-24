---
hide:
  - toc
---

# `install` Command

The `install` command configures the **runtime workspace** and the **database** for ScipionAPI.

It prepares the application to run, but it does **not** start runtime services.

!!! note "Scope"
    `install` handles runtime configuration, database setup/migrations, and admin-user creation. Service startup is a separate step (`start`) unless you use `provision`.

---

## Usage

```bash
./scripts/scipionapi install \
  --user "admin" \
  --email "admin@example.com" \
  --pass "changeMe"
```

---

## Required Options

| Option | Description |
|---|---|
| `--user` | Admin username |
| `--email` | Admin email |
| `--pass` | Admin password |

---

## What It Does

`install` typically performs the following actions:

1. Creates or updates `SCIPION_HOME`
2. Generates `.env`
3. Creates required folders (for example `logs/`, `projects/`)
4. Creates the PostgreSQL role and database *(local PostgreSQL only)*
5. Runs `alembic upgrade head`
6. Creates or updates the admin user

---

## Database Creation Behavior

### Automatic local database bootstrap

If:

- PostgreSQL is local, and
- the user has `sudo` privileges

then the role/database can be created automatically.

### Remote PostgreSQL (manual preparation)

If you use a remote PostgreSQL server:

- Create the database and role manually
- Ensure `.env` contains a valid `DATABASE_URL`
- Confirm connectivity/credentials before running `install`

---

## Idempotent Behavior

Running `install` again is generally safe and useful.

Typical behavior:

- Does **not** drop the database
- Updates admin credentials if needed
- Ensures migrations are applied/up to date

---

## Example

```bash
./scripts/scipionapi install \
  --user admin \
  --email admin@local \
  --pass changeMe
```

After `install`, start services:

```bash
./scripts/scipionapi start
```

---

## Common Issues

!!! warning "PostgreSQL authentication failed"
    Verify database credentials in `.env` and confirm the PostgreSQL role exists with the expected password.

!!! warning "Alembic migration error"
    Check connectivity and migration history state. A fresh empty database can help isolate migration-history problems.

!!! warning "Admin user not updated as expected"
    Re-run `install` with explicit options and inspect command output/logs for validation errors.

---

## Recommended Follow-Up Checks

After `install` and `start`:

```bash
./scripts/scipionapi status
curl http://localhost:8080/health
```

Expected response:

```json
{"status":"ok"}
```

---

## Navigation

<div style="display:flex; justify-content:space-between; align-items:center; width:100%; margin-top:2rem; gap:1rem;">
  <a href="../bootstrap/" style="text-decoration:none; display:inline-block;">
    ← Previous: bootstrap
  </a>
  <a href="../provision/" style="text-decoration:none; display:inline-block; margin-left:auto;">
    Next: provision →
  </a>
</div>
