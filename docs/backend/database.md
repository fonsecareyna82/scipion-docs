---
hide:
  - toc
---

# Database and Alembic Migrations

ScipionAPI persistence is built around:

- PostgreSQL
- SQLAlchemy ORM
- Alembic migrations

This combination supports relational data modeling, schema evolution, and predictable deployment upgrades.

---

## Database URL

Defined in `.env`:

```dotenv
DATABASE_URL=postgresql://user:pass@host:5432/db
```

!!! tip "Keep it consistent"
    If your deployment also uses split DB variables (`DATABASE_NAME`, `DATABASE_USER`, `DATABASE_PASS`), keep them aligned with `DATABASE_URL`.

---

## ORM Models

Models are located in:

```text
app/backend/models/
```

Typical entities include:

- `User`
- `Project`
- `Protocol`
- `ProjectShare`
- `Settings`
- `Tags`

!!! note "Model set may evolve"
    The exact list of models may change as features are added (for example new metadata or sharing/tagging entities).

---

## Alembic

Migration files are stored in:

```text
alembic/
```

Alembic tracks schema revisions and applies them in order.

---

## Apply Migrations

Run:

```bash
alembic upgrade head
```

This upgrades the database schema to the latest known revision.

!!! tip "Before production migrations"
    Always back up the database before applying migrations in production.

---

## Create a New Migration

Generate a migration:

```bash
alembic revision --autogenerate -m "description"
```

Then review the generated migration carefully before applying it.

!!! warning "Autogenerate is not infallible"
    Always inspect generated migrations for unexpected drops, type changes, defaults, or constraint operations.

---

## Migration Flow

```text
Model change → alembic revision → review migration → alembic upgrade
```

Recommended workflow:

1. Update ORM models
2. Generate migration
3. Review/edit migration if needed
4. Test on a local/staging DB
5. Apply to production (after backup)

---

## Best Practices

- Never edit **already applied** migrations in shared environments
- Test migrations on staging before production
- Back up the database before production migration
- Keep migration history linear and understandable when possible
- Review SQL/DDL implications for large tables

---

## Troubleshooting Migration State

If migration state becomes inconsistent:

- Use a fresh empty database to isolate schema issues
- Inspect the `alembic_version` table
- Confirm all migration files are present and in the expected order
- Check whether any applied migration was modified after deployment

!!! warning "Modified migration files"
    Editing an already applied migration can cause hard-to-debug state mismatches across environments.

---

## Operational Checks

Verify connectivity before migrations:

```bash
psql -U user -d db
```

Then run:

```bash
alembic upgrade head
```

After migration, validate expected tables:

```bash
psql -U user -d db -c "\dt"
```

---

## Navigation

<div style="display:flex; justify-content:space-between; align-items:center; width:100%; margin-top:2rem; gap:1rem;">
  <a href="../auth/" style="text-decoration:none; display:inline-block;">
    ← Previous: Authentication and JWT
  </a>
  <a href="../celery/" style="text-decoration:none; display:inline-block; margin-left:auto;">
    Next: Celery and Redis →
  </a>
</div>
