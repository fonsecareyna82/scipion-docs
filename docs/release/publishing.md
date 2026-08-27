# Publish a ScipionWeb Release

This page describes the **maintainer release workflow** for building and publishing the paired API and Web artifacts.

The current release command builds fresh artifacts by default and is also responsible for the release-server bookkeeping. Maintainers should not normally build the paired ZIPs manually or edit the remote `manifest.json` by hand.

---

## Required local source state

For the default release path you need:

- the ScipionAPI repository containing the release command
- the matching ScipionWeb repository
- matching versions declared by both packages
- a writable `--downloads-dir` for generated artifacts
- `npm` available for the Web build

ScipionWeb is resolved as a sibling of ScipionAPI by default. Use `--web-root` when needed.

The command creates:

```text
/path/to/release/files/
├── ScipionAPI-vX.Y.Z.zip
└── ScipionWeb-vX.Y.Z-dist.zip
```

`install.sh` does **not** need to be copied into this directory. The publisher takes the installer from the current ScipionAPI repository root.

!!! note "Existing ZIP workflow"
    If you intentionally want to publish already-built archives, use `--upload --no-build`. In that mode the paired ZIP files must already exist.

---

## 1. Validate the release locally

Before publication, make sure:

- API tests are green
- Web build succeeds
- API and Web versions match before running `release`
- `npm run build:web` succeeds
- the generated API ZIP extracts with the expected packaged layout
- the generated Web ZIP contains the expected `app/` content
- a clean or disposable installation can be provisioned
- an existing disposable installation can resolve/update to the release

Do not publish first and validate later.

---

## 2. Run the real remote dry run

```bash
./scripts/scipionapi release \
  --upload \
  --downloads-dir /path/to/release/files \
  --dry-run
```

This first builds fresh local API/Web artifacts, then validates the actual publication target. It connects to the configured release server, reads the current remote state, and prints the release plan without modifying remote files.

The dry run therefore has **no remote mutation**, but it can create or replace files in the local `--downloads-dir`.

The default publication target is:

```text
scipion@nolan.cnb.csic.es:/home/scipion/scipionfiles/downloads/scipion/scipionWeb/
```

The corresponding public endpoint is:

```text
https://scipion.cnb.csic.es/downloads/scipion/scipionWeb/
```

---

## 3. Review the release plan

Confirm at minimum:

- version
- local API ZIP
- local Web ZIP
- API SHA256
- Web SHA256
- installer path
- SSH login
- remote directory
- public URL
- whether the release will become `latest`
- whether replacement mode is enabled

If the target version already exists, the command refuses to continue unless `--force` is explicitly provided.

---

## 4. Publish

After a successful dry run:

```bash
./scripts/scipionapi release \
  --upload \
  --downloads-dir /path/to/release/files
```

The command asks for confirmation before the real upload.

---

## What happens to `manifest.json`

The publisher manages it automatically:

- **remote manifest exists** → download it, preserve previous release entries, add the new release
- **remote manifest does not exist** → create a new manifest containing the release being published
- compute file sizes and SHA256 values from the local ZIP files
- move `latest` to the new version unless `--no-latest` is used
- upload the manifest as the final publication step

A release directory does not need a local `manifest.json`.

---

## Safe server-side order

The publisher writes:

```text
ScipionAPI-vX.Y.Z.zip
ScipionWeb-vX.Y.Z-dist.zip
install.sh
manifest.json
```

`manifest.json` is intentionally last.

This means the public release metadata is updated only after the downloadable artifacts and installer are already in place.

---

## Replacing an existing release

Normal releases are immutable by default.

If a version is already present in the manifest or its ZIP filenames already exist remotely, publication stops.

To inspect an intentional replacement safely:

```bash
./scripts/scipionapi release \
  --upload \
  --version vX.Y.Z \
  --downloads-dir /path/to/release/files \
  --force \
  --dry-run
```

Only remove `--dry-run` when replacement is genuinely intended.

!!! danger "Treat `--force` as exceptional"
    Replacing already-published artifacts can break reproducibility and invalidate assumptions made by existing installations. Prefer a new patch version whenever possible.

---

## Publish without changing `latest`

```bash
./scripts/scipionapi release \
  --upload \
  --version vX.Y.Z \
  --downloads-dir /path/to/release/files \
  --no-latest
```

The release is added to the manifest but does not become the default target for `latest` installs/updates.

---

## 5. Verify through the public endpoint

After publication, confirm that the public release endpoint exposes:

```text
install.sh
manifest.json
ScipionAPI-vX.Y.Z.zip
ScipionWeb-vX.Y.Z-dist.zip
```

Also verify that `manifest.json`:

- still contains previous releases
- contains the new release entry
- contains the expected API/Web SHA256 values
- points `latest` to the intended version

---

## 6. Validate the user paths

Test both new-user and existing-user flows against the published release.

### New installation

```bash
wget https://scipion.cnb.csic.es/downloads/scipion/scipionWeb/install.sh
chmod +x install.sh
./install.sh --check-only
./install.sh --version vX.Y.Z
```

### Existing installation

```bash
./scripts/scipionapi update --version vX.Y.Z --dry-run
```

Then validate the real update in a disposable installation before considering the release complete.

---

## Release principle

The complete publishing model is:

> **Match API/Web versions → release builds the paired ZIPs → dry-run against Nolan → publish once → manifest updated automatically → validate install/update paths**

That is the supported maintainer workflow.
