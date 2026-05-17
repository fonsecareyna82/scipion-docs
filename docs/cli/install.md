---
hide:
  - toc
---

# `install` Command

The `install` command configures the **runtime workspace** and the **database** for ScipionAPI.

It prepares the application to run, but it does **not** start runtime services.

!!! note "Scope"
    `install` handles runtime configuration, database setup, migrations, and admin-user creation. Service startup is a separate step unless you use `provision`.

---

## Usage

```bash
./scripts/scipionapi install \
  --user "admin" \
  --email "admin@example.com" \
  --pass "changeMe"
```

---

## Mental model

Use `install` when:

- Python and dependencies are already prepared
- you want controlled installation steps
- you need to inspect database and runtime setup before starting services

A simple sequence is:

1. `bootstrap`
2. `install`
3. `start`

---

## What It Does

`install` typically performs the following actions:

1. creates or updates `SCIPION_HOME`
2. generates `.env`
3. creates required folders such as `logs/` and `projects/`
4. creates the PostgreSQL role and database in common local setups
5. runs `alembic upgrade head`
6. creates or updates the admin user

---

## Database Creation Behavior

### Automatic local database bootstrap

If PostgreSQL is local and the user has `sudo` privileges, the role and database can usually be created automatically.

### Remote PostgreSQL

If you use a remote PostgreSQL server:

- create the database and role manually
- ensure `.env` contains a valid `DATABASE_URL`
- confirm connectivity before running `install`

---

## What to do next

After `install`, continue with:

```bash
./scripts/scipionapi start
./scripts/scipionapi status
curl http://localhost:8080/health
```

---

## Idempotent Behavior

Running `install` again is generally safe.

Typical behavior:

- does **not** drop the database
- updates admin credentials if needed
- ensures migrations are applied and up to date

---

## Common Issues

!!! warning "PostgreSQL authentication failed"
    Verify database credentials in `.env` and confirm the PostgreSQL role exists with the expected password.

!!! warning "Alembic migration error"
    Check connectivity and migration history state. A fresh empty database can help isolate migration problems.

!!! warning "Admin user not updated as expected"
    Re-run `install` with explicit options and inspect output or logs for validation errors.
