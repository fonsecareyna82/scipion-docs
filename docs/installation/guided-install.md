# Guided Installation

The guided installer is the **recommended way to install ScipionWeb on a new Linux machine**.

It downloads a matched ScipionAPI + ScipionWeb release, verifies the published checksums, installs into a stable directory, runs the normal `provision` workflow, and records the installation so it can later be updated or removed safely.

---

## 1. Download the installer

Download the small standalone installer from the public ScipionWeb release endpoint:

```bash
wget https://scipion.cnb.csic.es/downloads/scipion/scipionWeb/install.sh
chmod +x install.sh
```

`install.sh` is intentionally standalone. You do **not** need to download the API or Web ZIP files yourself for the normal installation path.

---

## 2. Check the machine first

Before installing anything, run the prerequisite check:

```bash
./install.sh --check-only
```

The preflight validates the supported Linux runtime and checks the important installation dependencies together, including:

- `curl` or `wget`
- `unzip`
- `sudo`
- Conda and its base Python
- PostgreSQL client/server access
- Valkey client/server availability
- local PostgreSQL administrative access through `sudo -u postgres psql`

The installer **does not automatically install missing system packages**. If something is missing or misconfigured, it reports the problems and exits so the host can be fixed explicitly.

!!! tip "Run the preflight first"
    `--check-only` is the fastest way to distinguish a host prerequisite problem from a ScipionWeb installation problem.

---

## 3. Run the installer

For the normal interactive installation:

```bash
./install.sh
```

The installer asks for the required values, including:

- installation directory
- administrator username
- administrator email
- administrator password using a hidden prompt

The default installation directory is:

```text
$HOME/scipionweb
```

The default release is `latest`.

---

## What the installer does

A normal run performs the following high-level sequence:

1. validates system prerequisites
2. validates the target installation directory
3. downloads `manifest.json` from the public release server when available
4. resolves `latest` or the requested release version
5. downloads the matching ScipionAPI and ScipionWeb ZIP files
6. verifies SHA256 checksums published in the manifest
7. extracts ScipionAPI into the final installation directory
8. delegates environment, database, migrations, Web deployment, and startup to `scripts/scipionapi provision`
9. creates a `.scipionweb-installation` marker only after successful provisioning

The API and Web artifacts therefore come from the **same release entry** in `manifest.json`.

---

## Install a specific version

To install a specific published release:

```bash
./install.sh --version v4.0.1
```

Versions without the leading `v` are also normalized when valid:

```bash
./install.sh --version 4.0.1
```

Use `latest` to follow the `latest` value published in `manifest.json`.

---

## Choose another installation directory

```bash
./install.sh --install-dir /data/scipionweb
```

The installer deliberately refuses unsafe or ambiguous targets. In particular, it will not silently install over an unrelated non-empty directory or an existing ScipionWeb installation.

If an installation already exists, use its updater instead:

```bash
/path/to/scipionweb/scripts/scipionapi update
```

---

## API/Web port selection

You normally do not need to choose a port manually.

When `--api-port` is omitted, provisioning preserves an existing `API_PORT` when appropriate or selects an available port automatically and stores it in the installation environment.

To force a specific port:

```bash
./install.sh --api-port 39080
```

After installation, inspect the persisted value with:

```bash
grep '^API_PORT=' "$HOME/scipionweb/scipion_home/.env"
```

Adjust the path if you selected a custom installation directory.

---

## Non-interactive installation

For automation, provide the required values explicitly and supply the password through `SCIPIONWEB_ADMIN_PASSWORD`:

```bash
export SCIPIONWEB_ADMIN_PASSWORD='<admin-password>'

./install.sh \
  --non-interactive \
  --install-dir /data/scipionweb \
  --user admin \
  --email admin@example.org
```

Useful environment equivalents include:

```text
SCIPIONWEB_INSTALL_DIR
SCIPIONWEB_ADMIN_USER
SCIPIONWEB_ADMIN_EMAIL
SCIPIONWEB_ADMIN_PASSWORD
SCIPIONWEB_VERSION
SCIPIONWEB_DOWNLOAD_BASE_URL
SCIPIONWEB_API_PORT
SCIPIONAPI_CONDA_EXE
```

Avoid placing real passwords directly in shell command arguments.

---

## Verify the installation

From the installed directory, run:

```bash
./scripts/scipionapi status
./scripts/scipionapi doctor --quick
```

To inspect the selected port:

```bash
grep '^API_PORT=' scipion_home/.env
```

For deeper troubleshooting:

```bash
./scripts/scipionapi doctor
./scripts/scipionapi logs
```

---

## Failed installation behavior

If provisioning fails after installation has started, the installer preserves the installation directory for inspection instead of deleting potentially useful logs and state.

Look under:

```text
<installation-root>/scipion_home/logs/
```

Fix the reported problem and then decide whether to retry or clean up the incomplete installation.

---

## Updating later

A guided installation is updated from its installed ScipionAPI root:

```bash
./scripts/scipionapi update --dry-run
./scripts/scipionapi update
```

The updater preserves `SCIPION_HOME`, `.env`, projects, logs, and database data while replacing managed application files and the deployed Web bundle.

See [Upgrade / Reinstall Notes](upgrade.md) for details.

---

## Completely removing a guided installation

Preview the complete removal first. Run the command by absolute path so your shell is not left inside a directory that will be deleted:

```bash
/path/to/scipionweb/scripts/scipionapi uninstall --full --dry-run
```

Then, when the plan is correct:

```bash
/path/to/scipionweb/scripts/scipionapi uninstall --full
```

`--full` validates the guided installation marker, installation root, `SCIPION_HOME`, and expected ScipionAPI layout before destructive cleanup.

See the [uninstall CLI reference](../cli/uninstall.md) before using full removal on important systems.

---

## When to use the manual paths instead

Use the manual bundle/provision documentation when you intentionally need one of these scenarios:

- API-only deployment
- frontend hosted separately
- custom or remote PostgreSQL setup
- installation debugging
- development environments
- infrastructure-specific deployment automation

For normal new installations, prefer:

> **Prerequisites → `install.sh --check-only` → `install.sh`**
