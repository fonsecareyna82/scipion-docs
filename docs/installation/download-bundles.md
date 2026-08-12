---
hide:
  - toc
---

# Download and Extract Bundles

This page documents the **manual/advanced bundle workflow**.

For a normal new installation, you no longer need to download the ScipionAPI and ScipionWeb ZIP files yourself. Use the [Guided Installation](guided-install.md) instead:

```bash
wget https://scipion.cnb.csic.es/downloads/scipion/scipionWeb/install.sh
chmod +x install.sh
./install.sh
```

Use the manual bundle workflow when you intentionally want to inspect, test, archive, or provide the release ZIP files yourself.

---

## Official download endpoint

Published ScipionWeb release files live at:

[https://scipion.cnb.csic.es/downloads/scipion/scipionWeb/](https://scipion.cnb.csic.es/downloads/scipion/scipionWeb/)

The release endpoint normally contains:

```text
install.sh
manifest.json
ScipionAPI-vX.Y.Z.zip
ScipionWeb-vX.Y.Z-dist.zip
```

`manifest.json` records the available releases, their matching API/Web artifacts, SHA256 metadata, and the version identified as `latest`.

---

## Release artifact naming

The standard filenames are:

```text
ScipionAPI-vX.Y.Z.zip
ScipionWeb-vX.Y.Z-dist.zip
```

For example:

```text
ScipionAPI-v4.0.1.zip
ScipionWeb-v4.0.1-dist.zip
```

Keep the API and Web artifacts on the same release version unless you are intentionally performing a compatibility test.

---

## Recommended manual directory layout

A simple local workspace can look like:

```text
$HOME/scipionweb-manual/
├── ScipionAPI-v4.0.1.zip
└── ScipionWeb-v4.0.1-dist.zip
```

After extracting the API ZIP:

```text
$HOME/scipionweb-manual/
├── ScipionAPI-v4.0.1.zip
├── ScipionWeb-v4.0.1-dist.zip
└── <extracted-api-root>/
    ├── app/
    ├── scipionapi_cli/
    ├── scripts/
    ├── alembic/
    ├── pyproject.toml
    └── alembic.ini
```

The Web ZIP can remain compressed and be passed directly to `provision --web-dist`.

---

## Download a specific release

### Using wget

```bash
mkdir -p "$HOME/scipionweb-manual"
cd "$HOME/scipionweb-manual"

wget "https://scipion.cnb.csic.es/downloads/scipion/scipionWeb/ScipionAPI-v4.0.1.zip"
wget "https://scipion.cnb.csic.es/downloads/scipion/scipionWeb/ScipionWeb-v4.0.1-dist.zip"
```

### Using curl

```bash
mkdir -p "$HOME/scipionweb-manual"
cd "$HOME/scipionweb-manual"

curl -O "https://scipion.cnb.csic.es/downloads/scipion/scipionWeb/ScipionAPI-v4.0.1.zip"
curl -O "https://scipion.cnb.csic.es/downloads/scipion/scipionWeb/ScipionWeb-v4.0.1-dist.zip"
```

---

## Inspect the public manifest

To see which release is currently marked as `latest`:

```bash
curl -fsSL \
  https://scipion.cnb.csic.es/downloads/scipion/scipionWeb/manifest.json
```

A release entry conceptually contains:

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
      }
    }
  }
}
```

The guided installer and updater use this metadata automatically.

---

## Verify checksums manually

The normal guided installer verifies published SHA256 values automatically.

For a manual bundle workflow, compare the downloaded files with the values in `manifest.json`:

```bash
sha256sum ScipionAPI-v4.0.1.zip
sha256sum ScipionWeb-v4.0.1-dist.zip
```

Do not continue if the calculated checksum differs from the published checksum.

---

## Extract the API bundle

```bash
unzip ScipionAPI-v4.0.1.zip
```

The archive can either contain the API files at its root or inside one versioned directory. Locate the extracted ScipionAPI root and verify that it contains at least:

```text
app/
scipionapi_cli/
scripts/scipionapi
alembic/
pyproject.toml
alembic.ini
```

Make the wrapper executable if required:

```bash
chmod +x scripts/scipionapi
```

---

## Web bundle usage

For integrated mode, the Web ZIP can be passed directly to `provision`:

```bash
./scripts/scipionapi provision \
  --user admin \
  --email admin@example.org \
  --web-dist /path/to/ScipionWeb-v4.0.1-dist.zip
```

Manual extraction is optional. If you inspect the Web ZIP, its packaged content should resolve to a valid Vite `dist` directory containing the expected `index.html` and assets.

---

## When this manual path is appropriate

Use this workflow when:

- validating release artifacts before publication
- reproducing a packaging problem
- installing from an offline/local artifact mirror
- testing a specific API/Web bundle pair
- building custom deployment automation around `provision`
- hosting the frontend separately

For ordinary end-user installation, prefer the [Guided Installation](guided-install.md).

---

## Common mistakes

!!! warning "Using mismatched API and Web versions"
    Normal releases are published as pairs. Keep the versions aligned.

!!! warning "Skipping checksum verification in a manual install"
    The guided installer performs this automatically; a manual workflow must preserve the same trust boundary explicitly.

!!! warning "Looking for `scripts/scipionapi` inside the Web ZIP"
    The wrapper belongs to the ScipionAPI bundle. The Web ZIP contains the compiled frontend.

!!! warning "Treating this as the recommended new-user path"
    Manual downloading is supported, but `install.sh` is the normal installation entrypoint now.
