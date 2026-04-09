---
hide:
  - toc
---

# Upgrade Guide

This guide explains how to upgrade **ScipionAPI** and optionally **ScipionWeb** to a newer version.

!!! note "Recommended approach"
    Treat upgrades as controlled operations: **stop services**, **backup first**, **upgrade**, then **verify** before resuming normal usage.

---

## Upgrade mental model

A safe upgrade usually means preserving three things correctly:

1. the **runtime workspace** (`SCIPION_HOME`)
2. the **database state**
3. the **version alignment** between API and Web bundles

Most upgrade mistakes come from breaking one of those three.

---

## Typical upgrade flow

A common upgrade involves:

1. stopping services
2. backing up the database
3. extracting the new API bundle
4. reusing the existing `SCIPION_HOME`
5. running `provision` from the new bundle
6. optionally upgrading the Web bundle
7. verifying health, logs, and basic UI behavior

---

## 1. Stop services

From the current installation directory:

```bash
./scripts/scipionapi stop
./scripts/scipionapi status
```

Do not upgrade while services are still running.

---

## 2. Back up first

Before touching the new version, create a database backup.

This is the minimum safety step that makes rollback realistic.

If the deployment matters, also preserve the relevant runtime workspace state such as `.env` and project-related data.

---

## 3. Extract the new API bundle

Download the new bundle and extract it into a clean directory.

The important rule is: run the new version from the **new bundle directory**, but keep pointing to the **existing runtime workspace**.

---

## 4. Reuse existing `SCIPION_HOME`

Do **not** delete your existing runtime workspace.

That workspace usually contains:

- `.env`
- logs
- projects
- runtime configuration
- deployed web assets in integrated setups

Before proceeding, verify that the reused `.env` still points to the correct database and services.

---

## 5. Run `provision` again

From the **new API bundle directory**, run `provision` again.

The practical goal here is:

- refresh dependencies if needed
- apply migrations
- preserve data
- bring runtime services back under the new version

---

## 6. Upgrade the Web bundle if needed

If you are using integrated mode, pass the new Web bundle with `--web-dist` during the new `provision` run.

Keep API and Web versions aligned whenever possible.

---

## 7. Verify before calling it done

Check:

- `status`
- `/health`
- logs
- login
- project loading
- one basic workflow action in the UI

The point is not only that processes run, but that the application behaves normally again.

---

## Rollback thinking

If the upgrade fails badly, rollback usually means:

1. stop services
2. restore the database backup
3. go back to the previous API bundle
4. restart from the previously known-good state

Rollback is much safer when the backup was created before any migration step.

---

## Common upgrade mistakes

!!! warning "Running the new code against the wrong runtime workspace"
    Always verify which `SCIPION_HOME` the new bundle is actually using.

!!! warning "Upgrading code without verified backup"
    If migrations change the schema, rollback without a backup becomes much harder.

!!! warning "API and Web versions drift apart"
    Even if the UI loads, behavior may still be inconsistent when bundle versions do not match.
