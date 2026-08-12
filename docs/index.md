---
hide:
  - toc
---

# ScipionWeb Documentation

!!! info "What is ScipionWeb?"
    ScipionWeb is a web interface for managing **Scipion projects, protocols, outputs, viewers, plugins, and collaboration workflows** from the browser.

    This documentation helps installers, users, administrators, developers, and release maintainers find the right workflow without needing to understand the entire system first.

---

## What do you want to do?

### Install ScipionWeb

For a normal new Linux installation, use the guided installer. It checks prerequisites, resolves the matched API + Web release, verifies checksums, and provisions the integrated application.

1. [Installation Overview](installation/index.md)
2. [Prerequisites](installation/prerequisites.md)
3. [Guided Installation](installation/guided-install.md)

Fast path:

```bash
wget https://scipion.cnb.csic.es/downloads/scipion/scipionWeb/install.sh
chmod +x install.sh
./install.sh --check-only
./install.sh
```

### Use ScipionWeb

Start here if you already have access to a running ScipionWeb instance and want to work with projects, protocols, outputs, viewers, settings, plugins, or sharing.

1. [User Guide Overview](user-guide/index.md)
2. [First Login and Session Basics](user-guide/first-login.md)
3. [Projects](user-guide/projects.md)
4. [Protocol Execution](user-guide/protocols.md)
5. [Outputs and Viewers](user-guide/outputs.md)

### Update an installation

Existing installations should use the release-aware updater rather than running a fresh installer over the existing directory.

```bash
./scripts/scipionapi update --dry-run
./scripts/scipionapi update
```

See [Upgrade / Reinstall Notes](installation/upgrade.md).

### Get help or report something

1. [Support Overview](support/index.md)
2. [Ask a Question](support/ask-a-question.md)
3. [Report a Bug](support/report-a-bug.md)
4. [Known Issues and Workarounds](support/known-issues.md)

### Administer or troubleshoot an installation

1. [Configuration Overview](configuration/index.md)
2. [Environment Variables](configuration/env.md)
3. [Logs and PID Files](operations/logs-and-pids.md)
4. [Backup and Restore](operations/backup-restore.md)
5. [Security Notes](operations/security.md)

### Develop or extend ScipionWeb

1. [Command Line Reference](cli/index.md)
2. [Backend Overview](backend/index.md)
3. [Frontend Overview](frontend/index.md)
4. [Local Dev Workflow](development/local-workflow.md)
5. [Release and Packaging](release/packaging-strategy.md)

### Publish a release

Release maintainers should prepare the matching API/Web ZIP files and use the automated publisher:

```bash
./scripts/scipionapi release \
  --upload \
  --version vX.Y.Z \
  --downloads-dir /path/to/release/files \
  --dry-run
```

See [Publish a ScipionWeb Release](release/publishing.md).

---

## Recommended deployment mode

### Integrated Mode

For normal user-facing installations, ScipionAPI serves the compiled frontend and mounts the API under `/api`.

The API/Web port is runtime configuration. When no fixed port is requested, provisioning can select an available port automatically and persist it as `API_PORT` in `SCIPION_HOME/.env`.

After installation, inspect the actual value rather than assuming a fixed port:

```bash
grep '^API_PORT=' /path/to/scipionweb/scipion_home/.env
```

Then the normal URLs are conceptually:

```text
Web UI:   http://host:<API_PORT>/
API docs: http://host:<API_PORT>/api/docs
```

### Separate frontend/backend

Use this for advanced infrastructure where the frontend is hosted separately from ScipionAPI.

### API-only

Use this mainly for development, testing, or deployments where another service provides the frontend.

---

## Installation architecture in one line

The current user-facing installation model is:

> **`install.sh` → release manifest → paired API/Web downloads → checksum verification → `provision` → integrated ScipionWeb**

Advanced users can still download bundles and invoke `provision` manually when needed.

---

## What ScipionWeb provides

!!! abstract "Main capabilities"
    - Web-based project and protocol management
    - Output previews and specialized viewers
    - User authentication and sharing workflows
    - Plugin-related workflows
    - PostgreSQL persistence
    - Celery + Redis background task execution
    - Integrated API + Web deployment mode
    - CLI tools for installation, provisioning, updates, diagnostics, runtime control, release publication, and cleanup
