---
hide:
  - toc
---

# Download and Extract Bundles

This section explains how to download the official Scipion bundles and prepare the installation directory.

ScipionWeb is distributed as:

- **ScipionAPI bundle** (server)
- **ScipionWeb bundle** (compiled frontend, optional for integrated mode)

!!! note "Integrated mode"
    The **ScipionWeb** bundle is optional if you plan to run the API without serving the frontend from the same installation.

---

## Official Download Location

All versioned bundles are published at:

[https://scipion.cnb.csic.es/downloads/scipion/scipionWeb/](https://scipion.cnb.csic.es/downloads/scipion/scipionWeb/)

You should download:

- `ScipionAPI-<version>.zip`
- `ScipionWeb-<version>-dist.zip` *(optional)*

!!! tip "Version placeholder"
    Replace `<version>` with the actual published version number (for example, `3.0.0`).

---

## Recommended Directory Layout

We recommend installing everything under a dedicated directory:

```bash
$HOME/scipionweb/
```

After downloading (and extracting the API bundle), the layout should look like this:

```text
scipionweb/
├── ScipionAPI-<version>/
│   ├── scripts/
│   ├── scipionapi_cli/
│   ├── app/
│   ├── alembic/
│   └── pyproject.toml
│
└── ScipionWeb-<version>-dist.zip
```

!!! tip "Keep the Web ZIP file"
    If you are using **integrated mode**, you can pass the Web bundle ZIP directly to the installer with `--web-dist` (no manual extraction required).

---

## Download the Bundles

=== "Using wget"

    ```bash
    mkdir -p "$HOME/scipionweb"
    cd "$HOME/scipionweb"

    wget https://scipion.cnb.csic.es/downloads/scipion/scipionWeb/ScipionAPI-<version>.zip
    wget https://scipion.cnb.csic.es/downloads/scipion/scipionWeb/ScipionWeb-<version>-dist.zip
    ```

=== "Using curl"

    ```bash
    mkdir -p "$HOME/scipionweb"
    cd "$HOME/scipionweb"

    curl -O https://scipion.cnb.csic.es/downloads/scipion/scipionWeb/ScipionAPI-<version>.zip
    curl -O https://scipion.cnb.csic.es/downloads/scipion/scipionWeb/ScipionWeb-<version>-dist.zip
    ```

!!! warning "Optional Web bundle"
    If you do **not** need integrated mode, you may skip downloading `ScipionWeb-<version>-dist.zip`.

---

## Extract the API Bundle

```bash
unzip ScipionAPI-<version>.zip
cd ScipionAPI-<version>
```

You should now see the `scripts/scipionapi` wrapper in the extracted API directory.

---

## Web Bundle (Optional)

If you want **integrated mode** (the API serves the frontend):

- You do **not** need to extract the Web ZIP manually.
- You can pass it directly to the installer using `--web-dist`.

If you want to inspect it manually, extract it with:

```bash
unzip ScipionWeb-<version>-dist.zip
```

The extracted contents should include:

```text
dist/
  index.html
  assets/
```

!!! note "Manual inspection only"
    Manual extraction is useful for verification, but it is not required for a standard integrated installation.

---

## Verify Extraction

From inside the API directory, run:

```bash
ls -al
```

You should see at least:

- `scripts/`
- `scipionapi_cli/`
- `app/`
- `alembic/`
- `pyproject.toml`

!!! tip "Quick sanity check"
    If `scripts/scipionapi` is missing, verify that you extracted the correct ZIP file and that the extraction completed successfully.

---

<div style="display:flex; justify-content:space-between; align-items:center; width:100%; margin-top:2rem; gap:1rem;">
  <a href="../prerequisites/" style="text-decoration:none; display:inline-block;">
    ← Previous: Prerequisites
  </a>
  <a href="../provision/" style="text-decoration:none; display:inline-block; margin-left:auto;">
    Next: Provision →
  </a>
</div>
