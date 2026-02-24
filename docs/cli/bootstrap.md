---
hide:
  - toc
---

# `bootstrap` Command

The `bootstrap` command creates and prepares the **Conda environment** used by ScipionAPI.

It prepares the Python runtime and dependencies, but it does **not** perform installation tasks such as database creation or migrations.

## What `bootstrap` does *not* do

- It does **not** create the database
- It does **not** run Alembic migrations
- It does **not** start services

!!! note "Scope"
    `bootstrap` focuses only on preparing the Python execution environment.

---

## Usage

```bash
./scripts/scipionapi bootstrap
```

---

## What It Does

When executed, `bootstrap` typically:

1. Checks whether Conda is available
2. Creates the environment (default: `scipion4Web`) if it does not exist
3. Installs the Python version (default: `3.8`)
4. Upgrades `pip`
5. Installs dependencies from `requirements.txt`
6. Installs the package in editable mode (`pip install -e .`)

---

## Default Environment Name

Default Conda environment name:

```text
scipion4Web
```

This may be overridden in advanced setups (wrapper/env config, depending on version/workflow).

---

## When to Use `bootstrap`

Use `bootstrap` when you need to:

- Prepare a development environment
- Debug Conda or dependency issues
- Rebuild or refresh the Python runtime
- Perform a manual installation flow before `install`

=== "Manual / advanced flow"

    ```bash
    ./scripts/scipionapi bootstrap
    ./scripts/scipionapi install --user "admin" --email "admin@example.com" --pass "changeMe"
    ./scripts/scipionapi start
    ```

=== "One-shot alternative"

    ```bash
    ./scripts/scipionapi provision --user "admin" --email "admin@example.com" --pass "changeMe"
    ```

---

## Idempotent Behavior

Running `bootstrap` multiple times is generally safe.

Typical behavior:

- Does **not** recreate the environment if it already exists
- Reinstalls or refreshes dependencies if needed

---

## Example

```bash
./scripts/scipionapi bootstrap
```

After completion:

```bash
conda activate scipion4Web
scipionapi --help
```

---

## Troubleshooting

!!! warning "Conda not found"
    Confirm Conda is installed and available in `PATH`:

    ```bash
    conda --version
    ```

!!! warning "Environment created but command still fails"
    Activate the environment manually and test:

    ```bash
    conda activate scipion4Web
    scipionapi --help
    ```

!!! warning "Dependency installation errors"
    Re-run `bootstrap` and inspect terminal output. If the problem persists, verify Conda/Python compatibility and network access for package downloads.

---

## Next Step

After `bootstrap`, continue with:

```bash
./scripts/scipionapi install --user "admin" --email "admin@example.com" --pass "changeMe"
```

Or use the one-shot alternative:

```bash
./scripts/scipionapi provision --user "admin" --email "admin@example.com" --pass "changeMe"
```

---

## Navigation

<div style="display:flex; justify-content:space-between; align-items:center; width:100%; margin-top:2rem; gap:1rem;">
  <a href="../" style="text-decoration:none; display:inline-block;">
    ← Previous: CLI Reference Overview
  </a>
  <a href="../install/" style="text-decoration:none; display:inline-block; margin-left:auto;">
    Next: install →
  </a>
</div>
