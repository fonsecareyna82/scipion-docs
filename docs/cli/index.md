---
hide:
  - toc
---

# CLI Reference Overview

ScipionAPI provides a command-line interface to manage the lifecycle of a ScipionWeb deployment:

- environment bootstrapping
- runtime/database installation
- one-shot provisioning
- release-aware updates
- runtime service management
- diagnostics
- selective or full uninstall
- maintainer release publication

The main entrypoint for installed/package workflows is:

```bash
./scripts/scipionapi
```

If the package is already available inside the active environment, some commands can also be invoked through `scipionapi`, but the wrapper is the preferred operational entrypoint because it resolves the expected Conda/runtime context.

---

## New installations start with `install.sh`

The CLI is the engine behind installation, but most new users should not begin by manually invoking `bootstrap` or `provision`.

Use the public guided installer:

```bash
wget https://scipion.cnb.csic.es/downloads/scipion/scipionWeb/install.sh
chmod +x install.sh
./install.sh --check-only
./install.sh
```

The guided installer resolves/downloads the paired release and then delegates runtime setup to the CLI.

---

## Command mental model

- `bootstrap` → prepare Conda/Python/Scipion dependencies
- `install` → prepare `SCIPION_HOME`, `.env`, database, migrations, admin user
- `provision` → combine setup + optional Web deployment + startup
- `update` → move an existing installation to a published release
- `release` → publish a paired API/Web release as a maintainer
- `doctor` → inspect installation/runtime health without changing it
- `start`, `stop`, `restart`, `status`, `logs` → runtime operations
- `uninstall` → remove selected resources or a validated packaged installation
- `version` → show the installed CLI/release version

---

## Command groups

| Category | Commands | Purpose |
|---|---|---|
| Setup | `bootstrap`, `install`, `provision` | Prepare environment, runtime configuration, database, admin user, and optional Web deployment |
| Update | `update` | Update an existing installation from published release artifacts |
| Release | `release` | Maintainer-only publication of API/Web ZIPs, `install.sh`, and `manifest.json` |
| Runtime | `start`, `stop`, `restart`, `status`, `logs` | Manage API/Celery processes and inspect runtime logs |
| Diagnostics | `doctor` | Read-only checks for config, Conda, DB, Valkey, imports, and runtime state |
| Removal | `uninstall` | Selective runtime cleanup or protected full packaged-install removal |
| Info | `version` | Show installed ScipionAPI CLI version |

---

## Available commands

```text
bootstrap
install
provision
update
release
start
stop
restart
status
logs
doctor
version
```

The wrapper also handles:

```text
uninstall
uninstall-web
```

for runtime/full installation cleanup.

---

## Typical workflows

### New end-user installation

```bash
wget https://scipion.cnb.csic.es/downloads/scipion/scipionWeb/install.sh
chmod +x install.sh
./install.sh --check-only
./install.sh
```

### Existing installation update

```bash
./scripts/scipionapi update --dry-run
./scripts/scipionapi update
```

### Manual/advanced provision

```bash
./scripts/scipionapi provision \
  --user "admin" \
  --email "admin@example.org" \
  --web-dist /path/to/ScipionWeb-vX.Y.Z-dist.zip
```

### Daily runtime operations

```bash
./scripts/scipionapi status
./scripts/scipionapi doctor --quick
./scripts/scipionapi logs
./scripts/scipionapi restart
```

### Full guided uninstall preview

```bash
/path/to/scipionweb/scripts/scipionapi uninstall --full --dry-run
```

### Maintainer release publication

```bash
./scripts/scipionapi release \
  --upload \
  --version vX.Y.Z \
  --downloads-dir /path/to/release/files \
  --dry-run
```

---

## Admin password handling

For interactive setup commands, omit the password and let the CLI prompt using hidden input:

```bash
./scripts/scipionapi install \
  --user "admin" \
  --email "admin@example.org"
```

For automation, provide the password through an environment variable:

```bash
export SCIPIONAPI_ADMIN_PASSWORD='<admin-password>'

./scripts/scipionapi install \
  --user "admin" \
  --email "admin@example.org" \
  --password-env SCIPIONAPI_ADMIN_PASSWORD
```

`--pass` / `--password` remain available for compatibility but are not preferred because shell arguments may be retained in history or process inspection.

---

## Wrapper execution model

The `scripts/scipionapi` wrapper is more than a shortcut. Depending on the command, it helps:

- resolve the configured Conda executable/environment
- propagate `SCIPION_HOME`
- run the Python CLI in the expected environment
- protect full uninstall operations
- remove the Conda environment and installation root only at the appropriate final stage

For operational commands, use the wrapper unless the documentation explicitly says otherwise.

---

## Recommended reading order

### New installation

1. [Guided Installation](../installation/guided-install.md)
2. [doctor](doctor.md)
3. [Runtime Commands](runtime.md)
4. [Upgrade](update.md)

### Manual/advanced setup

1. [bootstrap](bootstrap.md)
2. [install](install.md)
3. [provision](provision.md)
4. [doctor](doctor.md)
5. [Runtime Commands](runtime.md)

### Maintainers

1. [Release Publisher](release.md)
2. [Publish a ScipionWeb Release](../release/publishing.md)
3. [Release Checklist](../release/checklist.md)

---

## Getting help

General help:

```bash
./scripts/scipionapi --help
```

Command help:

```bash
./scripts/scipionapi provision --help
./scripts/scipionapi update --help
./scripts/scipionapi release --help
./scripts/scipionapi doctor --help
./scripts/scipionapi uninstall --help
```

---

## Common mistakes

!!! warning "Starting a new user install with low-level CLI commands"
    `bootstrap`/`install`/`provision` remain supported, but the public `install.sh` is now the preferred end-user entrypoint.

!!! warning "Assuming a fixed API port"
    Provisioning can select a free port automatically. Read the persisted `API_PORT` instead of assuming `8080`.

!!! warning "Using `release` as an end-user command"
    `release` publishes to the Scipion download infrastructure and is for release maintainers.

!!! warning "Running `--full` from a Git checkout"
    Full uninstall is designed for verified packaged/guided installations. Use regular cleanup for development checkouts.
