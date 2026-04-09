---
hide:
  - toc
---

# Download and Extract Bundles

This page explains how to download the official Scipion bundles and prepare the installation directory.

ScipionWeb is distributed as:

- **ScipionAPI bundle** (server)
- **ScipionWeb bundle** (compiled frontend, optional for integrated mode)

!!! note "Integrated mode"
    The **ScipionWeb** bundle is optional if you plan to run the API without serving the frontend from the same installation.

---

## What you should download

You normally need:

- `ScipionAPI-<version>.zip`
- `ScipionWeb-<version>-dist.zip` if you want integrated mode

The most important rule here is version alignment: try to keep API and Web bundles on the same release whenever possible.

---

## Recommended directory layout

A clean local layout looks like this:

```text
$HOME/scipionweb/
├── ScipionAPI-<version>/
└── ScipionWeb-<version>-dist.zip
```

The Web ZIP can remain unextracted if you plan to pass it directly to `provision` using `--web-dist`.

---

## Download approach

Use a dedicated directory, keep the API bundle extractable, and keep the Web ZIP easy to reference from the installer.

Useful commands:

```bash
mkdir -p "$HOME/scipionweb"
cd "$HOME/scipionweb"
```

Then download the bundle files using your preferred tool.

---

## Extract the API bundle

```bash
unzip ScipionAPI-<version>.zip
cd ScipionAPI-<version>
```

At this point, the main thing to verify is that the API directory contains the wrapper script and the expected project structure.

---

## What to verify after extraction

Inside the extracted API directory, confirm that you can see expected files such as:

- `scripts/`
- `app/`
- `alembic/`
- `pyproject.toml`

A useful sanity check is making sure `scripts/scipionapi` exists before moving on.

---

## Web bundle usage

If you want **integrated mode**:

- keep the Web ZIP file available
- pass it directly to `provision` with `--web-dist`

Manual extraction of the Web bundle is optional and mostly useful for inspection.

---

## Common mistakes

!!! warning "Wrong or mismatched bundle versions"
    If API and Web versions drift too far apart, frontend and backend expectations may not align.

!!! warning "Extracting the wrong ZIP"
    Make sure you extracted the API bundle and not the Web bundle when looking for `scripts/scipionapi`.

!!! warning "Moving files before installation planning"
    Keep the layout simple until installation succeeds. Overcomplicated directory moves make debugging harder.
