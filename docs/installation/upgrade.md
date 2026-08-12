# Upgrade Guide

This guide explains how to upgrade an existing **ScipionWeb** installation using the built-in updater.

!!! success "Recommended upgrade path"
    Use `./scripts/scipionapi update` for normal upgrades. It preserves the runtime workspace and database data while updating managed ScipionAPI files and the deployed ScipionWeb bundle.

---

## Upgrade mental model

A safe upgrade preserves three things correctly:

1. the **runtime workspace** (`SCIPION_HOME`)
2. the **database state**
3. the **version alignment** between ScipionAPI and ScipionWeb

The updater is designed around that model. It updates application-managed files and Web assets while preserving `.env`, projects, logs, and database configuration.

---

## Recommended upgrade command

From the installed ScipionAPI root:

```bash
./scripts/scipionapi update
```

This resolves the release identified as `latest` from:

```text
https://scipion.cnb.csic.es/downloads/scipion/scipionWeb/manifest.json
```

The manifest identifies the matching API and Web artifacts and contains their checksum metadata.

---

## Preview before changing anything

Always inspect the update plan first on important installations:

```bash
./scripts/scipionapi update --dry-run
```

A dry run resolves the target release and prints the planned changes without replacing application files, applying migrations, deploying Web assets, or restarting runtime services.

---

## Upgrade to a specific version

```bash
./scripts/scipionapi update --version v4.0.1
```

The selected version must be present in the public release manifest or otherwise resolvable through the configured release source.

Standard paired artifacts use:

```text
ScipionAPI-v4.0.1.zip
ScipionWeb-v4.0.1-dist.zip
```

Keep API and Web versions aligned during normal upgrades.

---

## Reinstall the same version

If the installed API version already matches the target version, the updater may skip unnecessary replacement.

To intentionally reinstall the same version:

```bash
./scripts/scipionapi update --version v4.0.1 --force
```

Use this for controlled repair or release validation, not as the normal upgrade path.

---

## What the updater does

A normal update can perform the following sequence:

1. reads the existing installation environment and resolves `SCIPION_HOME`
2. resolves the target release from the public manifest
3. downloads the ScipionAPI and ScipionWeb ZIP files
4. verifies published SHA256 checksums when available
5. extracts and validates the release artifacts
6. creates a rollback backup of managed application files
7. stops runtime services
8. replaces managed ScipionAPI files
9. installs updated Python requirements/package state
10. applies Alembic migrations
11. redeploys the Web bundle when integrated mode is enabled
12. restarts services unless `--no-restart` was requested
13. writes update metadata back to `.env`
14. keeps the configured number of filesystem backups

The root `install.sh` is part of the managed API files, so an updated ScipionAPI release can also update the installer distributed with the installation.

---

## Runtime data preserved by update

Normal update is not an uninstall. It preserves:

- `SCIPION_HOME`
- `.env`
- projects
- logs
- database data/configuration
- user/admin state stored in the database
- the stable installation root

The updater replaces only the files and Web deployment areas it manages.

---

## Update metadata written to `.env`

After a successful update, metadata can include values such as:

```env
SCIPIONAPI_LAST_UPDATE_VERSION=v4.0.1
SCIPIONAPI_LAST_UPDATE_AT=20260812T143000Z
SCIPIONAPI_UPDATE_BASE_URL=https://scipion.cnb.csic.es/downloads/scipion/scipionWeb/
```

ScipionWeb can use backend system endpoints to expose the installed version and update availability.

---

## Update notifications in ScipionWeb

The backend can check the same public manifest used by the CLI updater.

Relevant endpoints include:

```http
GET /api/system/version
GET /api/system/update-check
```

The Web UI may inform users that a newer release is available, but the actual server update should be run from a controlled shell session.

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

Use `--api-only` and `--web-only` for controlled maintenance/debugging. Normal releases should move API and Web together.

---

## Backup and rollback thinking

The updater creates a filesystem rollback backup before replacing managed API files.

If an error occurs after that point, the updater can attempt to restore managed application files. Database migrations are different: database rollback still requires a database backup when schema changes matter.

For important/shared deployments:

1. back up PostgreSQL
2. run `update --dry-run`
3. apply the update during a controlled maintenance window
4. verify status, logs, login, project loading, and one basic workflow action

---

## Manual upgrade fallback

If the built-in updater cannot be used, a manual fallback remains possible:

1. stop services
2. back up PostgreSQL
3. obtain the new matched API/Web release pair
4. extract the new API bundle in a clean location
5. point it to the intended existing `SCIPION_HOME`
6. run `provision` with the matching Web bundle
7. verify runtime and application behavior

This path is more error-prone because the operator must preserve runtime paths, configuration, and version alignment explicitly.

---

## Release publication and `manifest.json`

Maintainers should **not** manually edit/upload `manifest.json` as part of the normal release process.

The release publisher now:

- downloads the current remote manifest when it exists
- preserves previous release entries
- adds the new release metadata and checksums
- publishes API ZIP, Web ZIP, and `install.sh`
- publishes `manifest.json` last

See [Publish a ScipionWeb Release](../release/publishing.md) and the [`release` CLI reference](../cli/release.md).

---

## Common upgrade mistakes

!!! warning "Skipping the dry run"
    On production or shared installations, inspect `./scripts/scipionapi update --dry-run` before the real update.

!!! warning "Forgetting database backups"
    The updater protects managed application files, but a database schema rollback still needs a database backup.

!!! warning "API and Web versions drift apart"
    Keep the paired release version aligned unless you are intentionally testing a compatibility scenario.

!!! warning "Treating an existing installation as a new install"
    Do not run a fresh `install.sh` over an existing installation directory. Use the installed `update` command.

---

## Guided-install users

If the original installation was created through the guided `install.sh`, nothing special is required for normal upgrades:

```bash
/path/to/scipionweb/scripts/scipionapi update --dry-run
/path/to/scipionweb/scripts/scipionapi update
```

The guided installation marker remains useful later for protected full uninstall; it does not change the normal update workflow.
