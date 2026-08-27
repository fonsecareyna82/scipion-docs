# Packaging Strategy

ScipionWeb is distributed as a **paired release** consisting of a ScipionAPI bundle and a compiled ScipionWeb bundle, with a small guided installer and a release manifest coordinating installation and updates.

The goal is a human-friendly deployment experience:

- no Git clone required for end users
- no manual API/Web version pairing
- checksum-verified downloads
- one guided installation entrypoint
- stable upgrades through the installed CLI
- safe, automated release publication

---

## High-level release model

A published release is represented by:

| Artifact | Purpose |
|---|---|
| `ScipionAPI-vX.Y.Z.zip` | Backend runtime, CLI, migrations, installer support, managed application files |
| `ScipionWeb-vX.Y.Z-dist.zip` | Pre-built frontend distribution |
| `install.sh` | Standalone guided installer used by new installations |
| `manifest.json` | Release index containing paired artifacts, checksums, sizes, and `latest` |

The API and Web ZIP files are versioned. `install.sh` and `manifest.json` are shared release-channel files and are updated as new releases are published.

---

## User installation model

The normal end-user path is:

```text
Download install.sh
        ↓
Run --check-only
        ↓
Run install.sh
        ↓
Resolve release from manifest.json
        ↓
Download matching API + Web ZIPs
        ↓
Verify SHA256
        ↓
Extract API into stable installation root
        ↓
Run provision
        ↓
Start integrated ScipionWeb
```

Users do not need to manually download or pair the release ZIPs for the standard path.

---

## Why two versioned bundles?

Keeping API and Web artifacts separate allows:

- integrated deployments where ScipionAPI serves the Web UI
- API-only deployments
- separately hosted static frontend deployments
- independent artifact validation/build pipelines
- clear separation between backend runtime and compiled frontend

The public release manifest still pairs the intended API and Web versions so normal installs/updates move together.

---

## Standard filenames

Use the canonical names:

```text
ScipionAPI-v4.0.1.zip
ScipionWeb-v4.0.1-dist.zip
```

In generic form:

```text
ScipionAPI-vX.Y.Z.zip
ScipionWeb-vX.Y.Z-dist.zip
```

Avoid introducing alternate naming conventions unless there is a strong compatibility reason. The installer, updater, publisher, and documentation all benefit from predictable filenames.

---

## ScipionAPI bundle layout

The API archive must extract to a valid ScipionAPI package root, either directly or through one enclosing directory.

The packaged API root is expected to contain key files/directories such as:

```text
app/
scipionapi_cli/
scripts/
alembic/
requirements.txt
pyproject.toml
alembic.ini
install.sh
README.rst
```

Runtime-generated state must not be bundled as release source content.

Do not package generated/runtime data such as:

```text
scipion_home/
.run/
.env
__pycache__/
```

---

## ScipionWeb bundle layout

The release command builds ScipionWeb with `npm run build:web`, reads the compiled `dist/app/` directory, and packages that content under an `app/` root in the Web ZIP.

Conceptually:

```text
app/
├── index.html
└── assets/
```

Runtime API configuration is injected/deployed by ScipionAPI; the Web bundle should not hardcode one installation-specific backend URL.

---

## `manifest.json`

The manifest is the release channel's source of truth for available paired versions.

Conceptually:

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
        "size": 654321
      },
      "generatedAt": "..."
    }
  }
}
```

The manifest enables:

- `latest` resolution
- API/Web pairing
- checksum verification
- update checks
- specific-version installation/update

Maintainers should not normally edit it by hand. The release publisher downloads the current remote manifest, preserves previous entries, adds the new release, and publishes the result automatically.

---

## Public distribution endpoint

The public download endpoint is:

```text
https://scipion.cnb.csic.es/downloads/scipion/scipionWeb/
```

The current release publisher writes to the corresponding server location through:

```text
scipion@nolan.cnb.csic.es:/home/scipion/scipionfiles/downloads/scipion/scipionWeb/
```

The physical SSH target and the public HTTP URL are separate concerns.

---

## Release build and publication model

A maintainer normally lets the release command build the paired archives directly from the ScipionAPI and ScipionWeb source trees.

The command first verifies that both packages declare the same version. An optional `--version vX.Y.Z` acts as an assertion against those package versions rather than overriding them.

Build locally:

```bash
./scripts/scipionapi release \
  --downloads-dir /path/to/release/files
```

Build fresh artifacts and validate the real remote publication plan:

```bash
./scripts/scipionapi release \
  --upload \
  --downloads-dir /path/to/release/files \
  --dry-run
```

After reviewing the plan, build and publish:

```bash
./scripts/scipionapi release \
  --upload \
  --downloads-dir /path/to/release/files
```

The publisher handles `install.sh`, `manifest.json`, checksums, remote-state validation, and safe upload ordering.

For intentionally pre-built artifacts, `--upload --no-build` preserves the older upload-only workflow.

---

## Safe publication ordering

Remote artifacts are published in this order:

```text
1. ScipionAPI ZIP
2. ScipionWeb ZIP
3. install.sh
4. manifest.json
```

`manifest.json` is the final step so a release is not advertised before its actual downloadable artifacts are in place.

Each upload is staged under a temporary remote name and moved into place atomically.

---

## Existing release protection

Published versions are protected by default.

The publisher refuses to overwrite a release if:

- the manifest already contains the target version, or
- the target API/Web filenames already exist remotely

`--force` is required for intentional replacement and should be exceptional. Prefer publishing a new patch version to preserve reproducibility.

---

## Versioning strategy

Use semantic-style release numbers:

```text
MAJOR.MINOR.PATCH
```

Published tags/artifacts normally include the leading `v`:

```text
v4.0.1
```

Typical meaning:

- **MAJOR** — breaking runtime/API/schema changes
- **MINOR** — backwards-compatible features
- **PATCH** — fixes and small compatible changes

---

## Supported deployment models

| Model | Description |
|---|---|
| Integrated | ScipionAPI serves both API and compiled ScipionWeb UI |
| Separate | ScipionAPI and Web static assets are hosted independently |
| API-only | Backend/runtime only, frontend provided elsewhere |

The guided installer targets the integrated model because it is the normal user-facing deployment.

---

## Design principles

- **No Git required** for normal installation
- **Release pairing is explicit** through the manifest
- **Checksums are verified** before installation/update
- **Ports are runtime configuration**, not a hardcoded release property
- **Runtime data is separate** from release source files
- **Updates preserve `SCIPION_HOME`**
- **Publication is atomic at the artifact level**
- **The manifest is published last**
- **Existing versions are protected from accidental replacement**
- **Full uninstall is restricted to verified packaged/guided installations**

---

## Summary

The current ScipionWeb release philosophy is:

> **Match API/Web versions → build and publish through the release CLI → install through `install.sh` → update through `scripts/scipionapi update`**

This keeps both the maintainer workflow and the end-user workflow deterministic and easy to audit.
