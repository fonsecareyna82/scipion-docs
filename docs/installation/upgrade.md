# Upgrade Guide

This guide explains how to upgrade **ScipionAPI** and **ScipionWeb** using the built-in updater.

!!! note "Recommended approach"
    Use `./scripts/scipionapi update` for normal upgrades. It keeps the existing runtime workspace, downloads the published release bundles, verifies checksums when available, creates a rollback backup, applies migrations, deploys the Web bundle when integrated mode is enabled, and restarts services.

---

## Upgrade mental model

A safe upgrade preserves three things correctly:

1. the **runtime workspace** (`SCIPION_HOME`)
2. the **database state**
3. the **version alignment** between ScipionAPI and ScipionWeb

The updater is designed around that model. It updates the application code and Web bundle while keeping the existing `.env`, projects, logs, database configuration, and runtime workspace.

---

## Recommended upgrade command

From the current ScipionAPI installation directory, run:

```bash
./scripts/scipionapi update
```

This resolves the latest available release from the public release manifest:

```text
https://scipion.cnb.csic.es/downloads/scipion/scipionWeb/manifest.json
```

The manifest tells the updater which ScipionAPI and ScipionWeb ZIP files belong to the latest release and provides checksum metadata when available.

---

## Preview before changing anything

Before running a real upgrade, use a dry run:

```bash
./scripts/scipionapi update --dry-run
```

A dry run prints the update plan without replacing files, installing dependencies, applying migrations, deploying Web assets, or restarting services.

Use this first when validating a production or shared instance.

---

## Upgrade to a specific version

To install a specific release instead of `latest`, pass the version explicitly:

```bash
./scripts/scipionapi update --version v4.0.1
```

The version should match the release bundle names published on the download server:

```text
ScipionAPI-v4.0.1.zip
ScipionWeb-v4.0.1-dist.zip
```

API and Web releases should normally move together. Avoid mixing unrelated API and Web versions unless you are debugging a very specific deployment issue.

---

## Reinstall the same version

If the installed API version already matches the target version, the updater may skip the API replacement unless forced.

To reinstall the same version:

```bash
./scripts/scipionapi update --version v4.0.1 --force
```

This is useful when validating a release package, repairing a partially updated install, or refreshing the Web bundle from a known release.

---

## What the updater does

A normal update performs these steps:

1. reads the existing `.env` and resolves `SCIPION_HOME`
2. resolves the target release from `manifest.json`
3. downloads the ScipionAPI and ScipionWeb ZIP files
4. verifies SHA256 checksums when provided by the manifest
5. extracts and validates the API source and Web distribution
6. creates a rollback backup under `SCIPION_HOME/updates/backups/`
7. stops runtime services
8. replaces managed API files
9. installs updated Python requirements and the ScipionAPI package
10. runs Alembic migrations
11. deploys the Web bundle when `SERVE_WEB=1`
12. restarts services unless `--no-restart` is used
13. writes update metadata back to `.env`
14. keeps only the configured number of backups

The command prints progress while running so users can see what the updater is doing.

---

## Runtime data preserved by update

The updater is not intended to erase runtime data. It keeps:

- `SCIPION_HOME`
- `.env`
- project data
- logs
- database configuration
- user/admin configuration
- existing runtime layout

The update replaces managed application files from the API bundle and deploys the Web bundle only through the configured installation paths.

---

## Update metadata written to `.env`

After a successful update, the updater stores metadata such as:

```env
SCIPIONAPI_LAST_UPDATE_VERSION=v4.0.1
SCIPIONAPI_LAST_UPDATE_AT=20260527T143000Z
SCIPIONAPI_UPDATE_BASE_URL=https://scipion.cnb.csic.es/downloads/scipion/scipionWeb/
```

ScipionWeb can use the backend system endpoints to display the installed version and whether a newer release is available.

---

## Update notifications in ScipionWeb

ScipionWeb can ask ScipionAPI whether a newer version exists. The backend checks the same public release manifest used by the CLI updater.

The relevant API endpoints are:

```http
GET /api/system/version
GET /api/system/update-check
```

When a newer version is available, the Home dashboard can show a notification and provide the command to run on the server, for example:

```bash
./scripts/scipionapi update --version v4.0.1
```

The Web UI should inform users, but the actual update should be run from the server shell during a controlled maintenance window.

---

## Backup and rollback thinking

The updater creates a filesystem rollback backup before replacing API files. If an error happens after the backup is created, it attempts to restore API-managed files.

However, database migrations are not automatically rolled back. For important deployments, always create a database backup before updating.

A safe rollback plan is still:

1. stop services
2. restore the database backup if migrations changed the schema
3. restore or reinstall the previous API/Web release
4. restart services
5. verify health, logs, login, project loading, and one basic workflow action

---

## Useful update options

```bash
./scripts/scipionapi update --dry-run
./scripts/scipionapi update --version v4.0.1
./scripts/scipionapi update --version v4.0.1 --force
./scripts/scipionapi update --api-only
./scripts/scipionapi update --web-only
./scripts/scipionapi update --no-restart
```

Use `--api-only` or `--web-only` only for controlled maintenance or debugging. Normal releases should keep API and Web versions aligned.

---

## Manual upgrade fallback

If the updater cannot be used, the old manual approach is still possible:

1. stop services
2. back up the database
3. extract the new API bundle into a clean directory
4. point the new bundle to the existing `SCIPION_HOME`
5. run `provision` from the new bundle
6. deploy the matching Web bundle
7. verify health, logs, login, project loading, and basic UI behavior

Manual upgrades are more error-prone because the operator must preserve runtime paths, deployment settings, and version alignment explicitly.

---

## Common upgrade mistakes

!!! warning "Updating without checking the plan first"
    On production or shared deployments, run `./scripts/scipionapi update --dry-run` before the real update.

!!! warning "Forgetting database backups"
    The updater backs up application files, but database rollback still requires a database backup.

!!! warning "API and Web versions drift apart"
    Keep ScipionAPI and ScipionWeb on the same release version unless you are intentionally testing a compatibility scenario.

!!! warning "Publishing `manifest.json` before ZIP files"
    During release publication, upload the ZIP files first and `manifest.json` last. Otherwise, users may see a latest version whose files are not yet available.
