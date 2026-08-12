# Prerequisites

Before installing **ScipionWeb**, prepare the target Linux machine with the system software, services, and permissions required by the guided installer and ScipionAPI runtime.

Ubuntu and Debian are the most straightforward environments for the documented commands.

!!! note "Recommended workflow"
    Install/verify the system dependencies first, then download `install.sh` and run:

    ```bash
    ./install.sh --check-only
    ```

    The guided installer checks the important prerequisites together before it downloads or provisions a release.

---

## What you need

A normal ScipionWeb deployment requires:

- **Linux**
- **Conda** (Miniconda or Anaconda)
- **PostgreSQL**
- **Redis**
- **sudo privileges** for the common local PostgreSQL bootstrap path
- **curl or wget**
- **unzip**
- enough disk space for the downloaded bundles, Conda environment, projects, logs, and updates

The installation flow uses these pieces to:

- create or reuse the Python environment
- configure the PostgreSQL database and role
- run migrations
- deploy the compiled Web UI
- run FastAPI and Celery services

---

## Supported operating system

### Recommended

- Ubuntu 22.04 / 24.04
- Debian 12+

!!! tip "Other Linux distributions"
    Other Linux distributions may work, but package names and service-management commands can differ.

---

## 1. Install Conda

ScipionAPI uses **Conda** to manage its Python environment.

### Check whether Conda is already installed

```bash
conda --version
```

If that prints a version number, continue to PostgreSQL.

### Install Miniconda

```bash
mkdir -p ~/Downloads
cd ~/Downloads
wget https://repo.anaconda.com/miniconda/Miniconda3-latest-Linux-x86_64.sh
bash Miniconda3-latest-Linux-x86_64.sh
```

Follow the installer prompts, then restart your terminal or run:

```bash
conda init bash
exec bash
```

Verify again:

```bash
conda --version
```

### If `conda` is not on PATH

For a standard Miniconda location:

```bash
export PATH="$HOME/miniconda3/bin:$PATH"
conda --version
```

For Anaconda, adjust the path accordingly.

The guided installer also accepts an explicit Conda executable:

```bash
export SCIPIONAPI_CONDA_EXE=/absolute/path/to/conda
```

!!! warning "Make PATH changes persistent"
    If `conda` only works after a manual `PATH` change, make that shell configuration persistent before continuing.

---

## 2. Install PostgreSQL

PostgreSQL stores the ScipionWeb/ScipionAPI persistent application state.

Install it on Ubuntu/Debian:

```bash
sudo apt update
sudo apt install -y postgresql postgresql-contrib
```

Enable and start it:

```bash
sudo systemctl enable --now postgresql
```

Verify it:

```bash
sudo systemctl status postgresql
sudo -u postgres psql -d postgres -c "SELECT 1;"
```

### Why sudo access matters

The standard local installation can create the PostgreSQL role and database through commands such as:

```bash
sudo -u postgres psql ...
```

The guided preflight validates this administrative path before installation.

If your deployment uses a custom or remote PostgreSQL setup, use the manual installation documentation instead of relying on the default local bootstrap.

---

## 3. Install Redis

Redis is used as the **Celery broker** and **result backend**.

Install it:

```bash
sudo apt update
sudo apt install -y redis-server
```

Enable and start it:

```bash
sudo systemctl enable --now redis-server
```

Verify it:

```bash
redis-cli ping
```

Expected output:

```text
PONG
```

---

## 4. Install the basic utilities

The guided installer needs a download tool and ZIP extraction support.

A practical Ubuntu/Debian package set is:

```bash
sudo apt update
sudo apt install -y \
  curl \
  wget \
  unzip \
  ca-certificates
```

Only one of `curl` or `wget` is required by the installer, but having both available is useful for administration and troubleshooting.

---

## 5. Ports

PostgreSQL and Redis normally use:

```text
PostgreSQL: 5432
Redis:      6379
```

ScipionWeb's API/Web port should **not** be assumed to be `8080`.

If no fixed port is requested, provisioning preserves an existing `API_PORT` where appropriate or selects a free port automatically and persists it in `SCIPION_HOME/.env`.

To inspect the standard service ports:

```bash
ss -ltnp | grep -E ':(5432|6379)\b'
```

If you plan to force a particular API/Web port, check that port explicitly before installation:

```bash
ss -ltnp | grep ':39080\b' || true
```

Then pass it to the installer:

```bash
./install.sh --api-port 39080
```

---

## 6. Permissions and installation directory

The installation user needs write access to the selected installation directory.

The guided installer defaults to:

```text
$HOME/scipionweb
```

It deliberately rejects unsafe or ambiguous targets, including unrelated non-empty directories and an already-installed ScipionWeb tree.

!!! caution "Avoid mixed ownership"
    Do not create parts of the ScipionWeb installation as `root` and then run the application as a regular user unless you intentionally manage ownership and permissions.

---

## 7. Run the official preflight

Once the basic packages are installed, download the public installer:

```bash
wget https://scipion.cnb.csic.es/downloads/scipion/scipionWeb/install.sh
chmod +x install.sh
```

Then run:

```bash
./install.sh --check-only
```

The check reports all detected missing software/configuration problems together. It validates, among other things:

- Linux runtime
- `curl` or `wget`
- `unzip`
- `sudo`
- Conda
- Conda base Python
- PostgreSQL client
- Redis client
- Redis server response
- PostgreSQL administrative access

If everything is ready, the installer exits successfully without installing ScipionWeb.

---

## Fast sanity check

These commands should all work before a normal installation:

```bash
conda --version
sudo systemctl is-active postgresql
sudo systemctl is-active redis-server
redis-cli ping
sudo -u postgres psql -d postgres -c "SELECT 1;"
```

Then confirm with the installer itself:

```bash
./install.sh --check-only
```

---

## Common prerequisite failures

### Conda not found

Try the actual Conda path or set `SCIPIONAPI_CONDA_EXE`:

```bash
export SCIPIONAPI_CONDA_EXE="$HOME/miniconda3/bin/conda"
./install.sh --check-only
```

### PostgreSQL is not running

```bash
sudo systemctl status postgresql
journalctl -u postgresql --no-pager -n 100
```

### Redis is not responding

```bash
sudo systemctl status redis-server
journalctl -u redis-server --no-pager -n 100
```

### PostgreSQL administrative access fails

Refresh sudo authentication:

```bash
sudo -v
sudo -u postgres psql -d postgres -c "SELECT 1;"
```

If your host intentionally does not allow this local administrative path, use the advanced/manual database setup instead.

---

## Next step

When `./install.sh --check-only` succeeds, continue with the [Guided Installation](guided-install.md).
