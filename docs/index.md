---
hide:
  - toc
---

# ScipionWeb Documentation

!!! info "What is ScipionWeb?"
    ScipionWeb is a web interface for managing **Scipion projects, protocols, outputs, viewers, plugins, and collaboration workflows** from the browser.

    This documentation helps installers, users, administrators, and developers find the right information without having to understand the whole system first.

---

## What do you want to do?

### Install ScipionWeb

Start here if you want to install ScipionWeb for users. The recommended path is **Integrated Mode**, where the API serves the Web UI from the same installation.

1. [Installation Overview](installation/index.md)
2. [Prerequisites](installation/prerequisites.md)
3. [Download and Extract Bundles](installation/download-bundles.md)
4. [Recommended Installation](installation/provision.md)

### Use ScipionWeb

Start here if you already have access to a running ScipionWeb instance and want to work with projects, protocols, outputs, viewers, settings, plugins, or sharing.

1. [User Guide Overview](user-guide/index.md)
2. [First Login and Session Basics](user-guide/first-login.md)
3. [Projects](user-guide/projects.md)
4. [Protocol Execution](user-guide/protocols.md)
5. [Outputs and Viewers](user-guide/outputs.md)

### Get help or report something

Start here if something does not work as expected, you need help, or you want to suggest an improvement.

1. [Support Overview](support/index.md)
2. [Ask a Question](support/ask-a-question.md)
3. [Report a Bug](support/report-a-bug.md)
4. [Known Issues and Workarounds](support/known-issues.md)

### Administer or troubleshoot an installation

Start here if you manage a ScipionWeb instance, configure runtime settings, review logs, handle backups, or investigate deployment problems.

1. [Configuration Overview](configuration/index.md)
2. [Environment Variables (`.env`)](configuration/env.md)
3. [Logs and PID Files](operations/logs-and-pids.md)
4. [Backup and Restore](operations/backup-restore.md)
5. [Security Notes](operations/security.md)

### Develop or extend ScipionWeb

Start here if you work on ScipionAPI, ScipionWeb, CLI commands, backend internals, frontend builds, or release packaging.

1. [Command Line Reference](cli/index.md)
2. [Backend Overview](backend/index.md)
3. [Frontend Overview](frontend/index.md)
4. [Local Dev Workflow](development/local-workflow.md)
5. [Release and Packaging](release/packaging-strategy.md)

---

## Recommended deployment mode

=== "Integrated Mode (recommended)"
    The backend serves the compiled frontend and mounts the API under `/api`.

    - Web UI: `http://host:8080/`
    - API docs: `http://host:8080/api/docs`

    This is the recommended mode when installing ScipionWeb for users.

=== "Separate frontend/backend"
    Use this when the frontend is hosted separately from the API, usually in more advanced infrastructure setups.

=== "API-only"
    Use this mainly for development, testing, or deployments where another service provides the frontend.

---

## Quick install example

```bash
./scripts/scipionapi provision \
  --user "admin" \
  --email "admin@example.com" \
  --pass "changeMe" \
  --web-dist "$HOME/scipionweb/ScipionWeb-<version>-dist.zip"
```

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
    - CLI tools for installation, provisioning, runtime control, and logs
