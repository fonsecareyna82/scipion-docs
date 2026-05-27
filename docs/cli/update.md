# update

The `update` command upgrades an existing ScipionAPI installation and, when integrated mode is enabled, the bundled ScipionWeb frontend.

Use it from the current ScipionAPI installation directory:

```bash
./scripts/scipionapi update
```

---

## Purpose

`update` is intended for existing deployments. It preserves the runtime workspace while replacing managed application files with the selected release.

It is designed to:

- read the current `.env`
- preserve `SCIPION_HOME`
- download the matching ScipionAPI and ScipionWeb release ZIPs
- verify checksums when available
- create a rollback backup
- stop services
- update API files
- install updated Python dependencies
- run database migrations
- deploy Web assets in integrated mode
- restart services

---

## Default update

Install the latest published release:

```bash
./scripts/scipionapi update
```

The updater resolves `latest` using:

```text
https://scipion.cnb.csic.es/downloads/scipion/scipionWeb/manifest.json
```

The manifest maps a release version to both ZIP files:

```text
ScipionAPI-vX.Y.Z.zip
ScipionWeb-vX.Y.Z-dist.zip
```

---

## Dry run

Preview what would happen without changing files:

```bash
./scripts/scipionapi update --dry-run
```

Use this before updating a production or shared instance.

---

## Specific version

Install a specific version:

```bash
./scripts/scipionapi update --version v4.0.1
```

This is useful when you want to pin the deployment to a known release instead of automatically using `latest`.

---

## Force reinstall

If the installed version already matches the target version, use `--force` to reinstall it:

```bash
./scripts/scipionapi update --version v4.0.1 --force
```

This can be useful when validating release packages, repairing a partial update, or redeploying the Web bundle from the same release.

---

## API-only or Web-only updates

Normal releases should keep ScipionAPI and ScipionWeb aligned. For controlled maintenance or debugging, you may update only one side:

```bash
./scripts/scipionapi update --api-only
./scripts/scipionapi update --web-only
```

Use these options carefully. A mismatched API/Web pair may load but still behave incorrectly.

---

## Do not restart automatically

To leave services stopped after the update:

```bash
./scripts/scipionapi update --no-restart
```

This is useful if you want to inspect files, logs, or migrations before bringing the instance back online.

---

## Release source

By default, updates are resolved from:

```text
https://scipion.cnb.csic.es/downloads/scipion/scipionWeb/
```

The base URL can be configured through:

```env
SCIPIONAPI_UPDATE_BASE_URL=https://scipion.cnb.csic.es/downloads/scipion/scipionWeb/
```

This is useful for mirrors, staging release directories, or private deployments.

---

## Manifest contract

The updater expects a `manifest.json` similar to:

```json
{
  "latest": "v4.0.1",
  "releases": {
    "v4.0.1": {
      "api": {
        "file": "ScipionAPI-v4.0.1.zip",
        "sha256": "...",
        "size": 123456
      },
      "web": {
        "file": "ScipionWeb-v4.0.1-dist.zip",
        "sha256": "...",
        "size": 123456
      }
    }
  }
}
```

The manifest should be cumulative. When publishing `v4.0.1`, keep previous releases such as `v4.0.0` in the `releases` object.

---

## Generated release manifest

Maintainers can generate or update the manifest with:

```bash
python scripts/update_release_manifest.py \
  --version v4.0.1 \
  --downloads-dir /path/to/releases
```

The release directory should contain:

```text
manifest.json
ScipionAPI-v4.0.1.zip
ScipionWeb-v4.0.1-dist.zip
```

If `manifest.json` already exists, the script preserves previous releases and adds or replaces the selected version.

---

## Publication order

When publishing a new release:

1. create `ScipionAPI-vX.Y.Z.zip`
2. create `ScipionWeb-vX.Y.Z-dist.zip`
3. generate or update `manifest.json`
4. upload both ZIP files
5. upload `manifest.json` last

Uploading `manifest.json` last prevents users from seeing a latest version whose ZIP files are not yet available.

---

## Example production flow

```bash
./scripts/scipionapi update --dry-run
./scripts/scipionapi update --version v4.0.1
./scripts/scipionapi status
./scripts/scipionapi doctor --quick
./scripts/scipionapi logs
```

Then verify the Web UI:

- login
- project list
- project loading
- one basic workflow or protocol action

---

## Related endpoints

ScipionAPI exposes system endpoints that ScipionWeb can use to show update information:

```http
GET /api/system/version
GET /api/system/update-check
```

These endpoints are informational. They do not run the update automatically.

---

## Related pages

- [Upgrade Guide](../../installation/upgrade/)
- [Runtime Commands](../runtime/)
- [Backup and Restore](../../operations/backup-restore/)
- [Release Checklist](../../release/checklist/)
