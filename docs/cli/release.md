# Release Publisher

The `release` command publishes a paired **ScipionAPI + ScipionWeb** release to the official download server.

It is intended for release maintainers and replaces the old manual workflow of copying ZIP files, calculating checksums, editing `manifest.json`, and uploading the manifest by hand.

---

## Minimal release command

If the release directory contains the correctly named API and Web ZIP files:

```bash
./scripts/scipionapi release \
  --upload \
  --version v4.0.1 \
  --downloads-dir /path/to/release/files
```

The expected default filenames are:

```text
ScipionAPI-v4.0.1.zip
ScipionWeb-v4.0.1-dist.zip
```

Only those two ZIP files need to be present in `--downloads-dir`.

`install.sh` is taken automatically from the current **ScipionAPI repository root**. You do not need to copy it into the release directory.

---

## Always run a dry run first

```bash
./scripts/scipionapi release \
  --upload \
  --version v4.0.1 \
  --downloads-dir /path/to/release/files \
  --dry-run
```

A release dry run is read-only with respect to the remote server, but it is not purely local. It deliberately:

- validates the local release files
- connects to the configured release server over SSH
- validates the remote release directory
- downloads the current remote `manifest.json` when present
- checks whether the version or target ZIP filenames already exist remotely
- calculates the release metadata and SHA256 values
- prints the resolved publication plan

It does **not** upload or replace remote files.

---

## Official defaults

The current ScipionWeb release publisher defaults to:

```text
SSH login:
scipion@nolan.cnb.csic.es

Remote directory:
/home/scipion/scipionfiles/downloads/scipion/scipionWeb

Public URL:
https://scipion.cnb.csic.es/downloads/scipion/scipionWeb/
```

These values can be overridden with:

```text
--login
--remote-dir
--base-url
```

or the corresponding environment variables:

```text
SCIPIONWEB_RELEASE_LOGIN
SCIPIONWEB_RELEASE_REMOTE_DIR
SCIPIONWEB_RELEASE_BASE_URL
```

---

## `manifest.json` is automatic

You do **not** prepare `manifest.json` manually before publishing.

The publisher follows this logic:

1. try to download the current remote `manifest.json`
2. if it exists, preserve its existing `releases` entries
3. add or update the requested release entry locally
4. calculate the API/Web SHA256 values and file sizes
5. update `latest` unless `--no-latest` was requested
6. publish the resulting manifest only after the release files are in place

If no remote `manifest.json` exists, the command creates a new one containing the release being published.

This means a normal release directory only needs:

```text
ScipionAPI-vX.Y.Z.zip
ScipionWeb-vX.Y.Z-dist.zip
```

---

## Safe publication order

The remote publication order is intentionally:

```text
1. ScipionAPI ZIP
2. ScipionWeb ZIP
3. install.sh
4. manifest.json
```

Each upload is staged through a temporary remote filename and moved into place atomically.

Publishing `manifest.json` last is important: users should never see a newly advertised release until its downloadable artifacts are already available.

---

## Existing versions are protected

Without `--force`, the publisher refuses to replace a version when either:

- the remote manifest already contains that version, or
- the target release ZIP filenames already exist remotely

Example refusal:

```text
Release already appears to be published (...). Use --force only when intentionally replacing it.
```

This is a safety feature, not a publication failure.

---

## Previewing an already-published version

If you intentionally want to inspect the complete plan for an existing version without changing anything, combine `--force` with `--dry-run`:

```bash
./scripts/scipionapi release \
  --upload \
  --version v4.0.0 \
  --downloads-dir /path/to/release/files \
  --force \
  --dry-run
```

!!! warning "Do not remove `--dry-run` casually"
    `--force` without `--dry-run` allows intentional replacement of an already-published version. Use it only when replacement is genuinely required.

---

## Publishing without changing `latest`

By default, the new release becomes the manifest's `latest` version.

To publish a release entry without moving `latest`:

```bash
./scripts/scipionapi release \
  --upload \
  --version v4.0.1 \
  --downloads-dir /path/to/release/files \
  --no-latest
```

This is useful for controlled publication or pre-release scenarios where the release should exist but should not yet become the default update target.

---

## Non-interactive confirmation

The command asks for confirmation before a real upload.

For controlled automation:

```bash
./scripts/scipionapi release \
  --upload \
  --version v4.0.1 \
  --downloads-dir /path/to/release/files \
  --yes
```

Use `--yes` only in release automation where the preceding validations are already trusted.

---

## Custom artifact paths

If the filenames do not follow the normal convention, use:

```text
--api-file
--web-file
```

The standard naming convention is strongly preferred because the installer, updater, server directory, and documentation all become easier to reason about.

---

## Post-publication verification

After uploading the artifacts and publishing `manifest.json`, the command checks the public manifest URL and verifies that the published release can be observed through HTTP.

A temporary visibility problem at the public endpoint is reported as a warning rather than pretending verification succeeded.

For an important release, also verify manually that:

```text
manifest.json
ScipionAPI-vX.Y.Z.zip
ScipionWeb-vX.Y.Z-dist.zip
install.sh
```

are reachable from the public release endpoint.

---

## Recommended maintainer workflow

```bash
# 1. Build the matching ZIP files.

# 2. Validate the real remote state and release plan.
./scripts/scipionapi release \
  --upload \
  --version vX.Y.Z \
  --downloads-dir /path/to/release/files \
  --dry-run

# 3. Publish after reviewing the plan.
./scripts/scipionapi release \
  --upload \
  --version vX.Y.Z \
  --downloads-dir /path/to/release/files
```

That is the normal release path. Manual editing of the remote manifest should not be part of routine publication.
