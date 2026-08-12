# Uninstall

The wrapper provides a safe ScipionWeb/ScipionAPI uninstall path:

```bash
./scripts/scipionapi uninstall
```

There are two distinct modes:

- **runtime cleanup**: remove selected runtime resources while keeping the ScipionAPI installation directory
- **full uninstall**: completely remove a packaged/guided installation after validated cleanup

Use `--dry-run` before destructive operations.

---

## Preview the cleanup plan

```bash
./scripts/scipionapi uninstall --dry-run
```

A dry run prints what would be stopped or removed without deleting resources.

For a full guided uninstall, prefer an absolute command path because the installation directory itself will disappear:

```bash
/path/to/scipionweb/scripts/scipionapi uninstall --full --dry-run
```

---

## Regular uninstall

Without `--full`, the repository/installation root is preserved.

Available cleanup controls include:

```text
--keep-database
--keep-database-role
--keep-web-dist
--remove-scipion-home
--remove-conda-env
--keep-conda-env
```

This mode is useful for development checkouts, troubleshooting, or selective runtime cleanup.

Example: remove deployed Web assets and PostgreSQL resources while leaving the ScipionAPI checkout itself in place:

```bash
./scripts/scipionapi uninstall
```

The command asks for confirmation unless `--yes`/`-y` or `--dry-run` is used.

---

## Full uninstall for guided installations

Installations created successfully by the public `install.sh` carry a marker file:

```text
.scipionweb-installation
```

That marker records the guided installation root and expected `SCIPION_HOME`.

Preview full removal:

```bash
/path/to/scipionweb/scripts/scipionapi uninstall --full --dry-run
```

Then run the real removal:

```bash
/path/to/scipionweb/scripts/scipionapi uninstall --full
```

A full uninstall removes the installation resources, including:

- running API/Celery runtime state
- configured PostgreSQL database
- configured PostgreSQL role when applicable
- deployed Web assets
- `SCIPION_HOME`
- the ScipionWeb Conda environment
- the installation root itself

---

## Full uninstall safety checks

`--full` is deliberately restrictive.

Before deleting the installation root, it verifies the installation identity and refuses unsafe layouts. Guided installations are checked for:

- supported marker format
- `INSTALL_TYPE=guided`
- marker `INSTALL_ROOT` matching the actual installation root
- marker `SCIPION_HOME` matching `<installation-root>/scipion_home`
- expected ScipionAPI files such as `pyproject.toml`, `alembic.ini`, and `scripts/scipionapi`
- unsafe roots such as `/` or the user's home directory
- symbolic-link or externally resolved `SCIPION_HOME` scenarios

The root is validated again before the final directory removal.

These checks are intentional safeguards against deleting an unrelated directory.

---

## Why `--full` cannot keep selected resources

A full uninstall represents complete removal, so it cannot be combined with:

```text
--keep-database
--keep-database-role
--keep-web-dist
```

If you need to preserve selected resources, use the regular uninstall mode instead.

---

## Older packaged installations

Older ZIP-based installations created before the guided installation marker existed can use:

```bash
/path/to/scipionweb/scripts/scipionapi uninstall \
  --full \
  --legacy-install \
  --dry-run
```

If the dry-run plan is correct:

```bash
/path/to/scipionweb/scripts/scipionapi uninstall \
  --full \
  --legacy-install
```

Legacy full uninstall is accepted only when the expected packaged ScipionAPI layout can be verified.

!!! danger "Git checkouts are not legacy packaged installs"
    `--full --legacy-install` explicitly refuses Git checkouts. For a development repository, use regular uninstall and remove the repository manually only if that is actually intended.

---

## Non-interactive cleanup

To skip the confirmation prompt:

```bash
./scripts/scipionapi uninstall --yes
```

For full removal:

```bash
/path/to/scipionweb/scripts/scipionapi uninstall --full --yes
```

Use `--yes` only after the target and cleanup plan have already been reviewed.

---

## Recommended removal workflow

```bash
# 1. Inspect the exact plan.
/path/to/scipionweb/scripts/scipionapi uninstall --full --dry-run

# 2. Review database, SCIPION_HOME, Conda env, and installation root carefully.

# 3. Run the real uninstall only when the resolved paths are correct.
/path/to/scipionweb/scripts/scipionapi uninstall --full
```

For shared or important deployments, back up anything that must be retained before running full removal.
