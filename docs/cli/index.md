---
hide:
  - toc
---

# CLI Reference Overview

ScipionAPI provides a command-line interface (CLI) to manage the full lifecycle of a deployment:

- environment bootstrapping
- runtime and database installation
- one-shot provisioning
- service lifecycle management
- logs and runtime status inspection

The main entrypoint is the wrapper script:

```bash
./scripts/scipionapi
```

If the package is already available inside the active environment, you may also use:

```bash
scipionapi
```

!!! note "Recommended usage"
    For most users, the wrapper script (`./scripts/scipionapi`) is the safest option because it helps ensure the command runs in the expected Conda environment.

---

## How to think about the CLI

A simple mental model is:

- `bootstrap` prepares Python and dependencies
- `install` prepares runtime configuration and database state
- `provision` combines setup into one guided path
- `start`, `stop`, `restart`, `status`, and `logs` manage the running services

If you remember that split, the CLI becomes much easier to understand.

---

## Command Groups

| Category | Commands | Purpose |
|---|---|---|
| Environment | `bootstrap` | Create or refresh the Conda runtime and install Python dependencies |
| Installation | `install`, `provision` | Configure runtime workspace, database, admin user, and optional web deployment |
| Runtime | `start`, `stop`, `restart`, `status`, `logs` | Manage API/Celery processes and inspect runtime logs |

!!! tip "Fastest path"
    If you want a complete setup in one command, use **`provision`**.

---

## Available Commands

- `bootstrap`
- `install`
- `provision`
- `start`
- `stop`
- `restart`
- `status`
- `logs`

---

## Typical Workflows

=== "Recommended (one-shot setup)"

    ```bash
    ./scripts/scipionapi provision \
      --user "admin" \
      --email "admin@example.com" \
      --pass "changeMe"
    ```

=== "Step-by-step (advanced or debugging)"

    ```bash
    ./scripts/scipionapi bootstrap
    ./scripts/scipionapi install --user "admin" --email "admin@example.com" --pass "changeMe"
    ./scripts/scipionapi start
    ```

=== "Daily runtime operations"

    ```bash
    ./scripts/scipionapi status
    ./scripts/scipionapi logs
    ./scripts/scipionapi restart
    ```

---

## Execution Model

The wrapper script typically:

1. detects or prepares the Conda environment
2. runs the CLI inside that environment
3. ensures Python dependencies are available for execution

That is why using the wrapper is often more reliable than invoking scattered commands manually.

---

## Recommended Reading Order

### First-time install

1. [provision](provision/)
2. [runtime commands](runtime/)
3. [logs and PID files](../operations/logs-and-pids/)

### Manual or debugging-oriented setup

1. [bootstrap](bootstrap/)
2. [install](install/)
3. [runtime commands](runtime/)
4. [Backend Troubleshooting](../backend/troubleshooting/)

---

## Getting Help

### General help

```bash
./scripts/scipionapi --help
```

### Command-specific help

```bash
./scripts/scipionapi provision --help
./scripts/scipionapi install --help
./scripts/scipionapi start --help
```

---

## Common Mistakes

!!! warning "Mixing `provision` and partial manual steps without checking state"
    `provision` is idempotent in common cases, but always confirm your runtime paths and database target before re-running commands.

!!! warning "Running commands outside the expected project directory"
    Use the wrapper from the API bundle directory so relative paths resolve correctly.

!!! warning "Forgetting to inspect logs after startup"
    A service can start and still fail shortly after. Use `status` and `logs` to verify runtime health.

---

## Navigation

<div style="display:flex; justify-content:space-between; align-items:center; width:100%; margin-top:2rem; gap:1rem;">
  <a href="../configuration/" style="text-decoration:none; display:inline-block;">
    ← Previous: Configuration Overview
  </a>
  <a href="bootstrap/" style="text-decoration:none; display:inline-block; margin-left:auto;">
    Next: bootstrap →
  </a>
</div>
