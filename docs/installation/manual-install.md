---
hide:
  - toc
---

# Manual Installation

This guide describes how to install **ScipionAPI** manually, step by step.

Use this method if:

- You prefer full control over the installation
- You are using a remote PostgreSQL server
- You do not want automatic database bootstrap
- You are debugging or developing locally

!!! note "Manual vs one-shot provisioning"
    If you want the fastest setup path, use [`provision`](../provision/). This page is intended for **controlled, step-by-step installation**.

---

## Overview

Manual installation typically consists of:

1. Creating a Conda environment
2. Installing Python dependencies
3. Configuring PostgreSQL manually
4. Running Alembic migrations
5. Creating the admin user
6. Starting API and Celery services

!!! tip "When this workflow is best"
    Manual installation is ideal for development, CI, debugging migration issues, and deployments that use a pre-existing or remote PostgreSQL server.

---

## 1. Create Conda Environment

Run the following commands from inside the extracted **ScipionAPI** directory:

```bash
conda create -n scipion4Web python=3.8 -y
conda activate scipion4Web
```

Upgrade `pip`:

```bash
python -m pip install --upgrade pip
```

!!! warning "Shell activation"
    If `conda activate` fails, initialize Conda first (`conda init bash`) and restart your shell.

---

## 2. Install Dependencies

### Install requirements

```bash
pip install -r requirements.txt
```

### Install the package in editable mode

```bash
pip install -e .
```

### Verify the CLI is available

```bash
scipionapi --help
```

!!! tip "Editable install"
    `pip install -e .` is recommended for local development because code changes are picked up without reinstalling the package.

---

## 3. Create PostgreSQL Database Manually

If you are not using automatic database bootstrap, create the PostgreSQL role and database manually.

### Login as `postgres`

```bash
sudo -u postgres psql
```

### Create role

```sql
CREATE USER scipion_user WITH PASSWORD 'yourStrongPassword';
```

### Create database

```sql
CREATE DATABASE scipion_db OWNER scipion_user;
```

### Grant privileges

```sql
GRANT ALL PRIVILEGES ON DATABASE scipion_db TO scipion_user;
```

### Exit `psql`

```sql
\q
```

!!! warning "Credentials"
    Use a strong password and store it securely. Avoid committing real credentials to repositories or shared files.

---

## 4. Configure Environment (`.env`)

### Create the runtime workspace

```bash
mkdir -p scipion_home
```

### Create the environment file

Create:

```text
scipion_home/.env
```

Example content:

```dotenv
DATABASE_URL=postgresql://scipion_user:yourStrongPassword@localhost:5432/scipion_db
DATABASE_NAME=scipion_db
DATABASE_USER=scipion_user
DATABASE_PASS=yourStrongPassword
SECRET_KEY=generate_a_secure_random_key

API_HOST=0.0.0.0
API_PORT=8080

BROKER_URL=redis://localhost:6379/0
LOGS_PATH=scipion_home/logs
PROJECTS_PATH=scipion_home/projects
```

### Export `SCIPION_HOME`

```bash
export SCIPION_HOME="$(pwd)/scipion_home"
```

!!! tip "Persist for future sessions"
    Add the `SCIPION_HOME` export to your shell profile (for example `~/.bashrc`) if you will run ScipionAPI regularly from the same installation.

---

## 5. Run Alembic Migrations

### Load environment variables (export them to child processes)

If your `.env` file uses `KEY=value` lines (without `export`), use:

```bash
set -a
source scipion_home/.env
set +a
```

### Run migrations

```bash
alembic upgrade head
```

This creates all required tables.

### Verify tables were created

```bash
psql -U scipion_user -d scipion_db -c "\dt"
```

!!! note "Database connection errors"
    If migration fails, verify PostgreSQL is running, credentials are correct, and the database is reachable from the current machine.

---

## 6. Create Admin User

Run the install command (without database bootstrap):

```bash
scipionapi install \
  --user "admin" \
  --email "admin@example.com" \
  --pass "changeMe"
```

This will:

- Ensure tables exist
- Create or update the admin user

!!! warning "Password handling"
    As with `provision`, CLI passwords may be stored in shell history. Use a temporary password if needed and rotate it after first login.

---

## 7. Start Services Manually

=== "Start API"

    ```bash
    uvicorn app.backend.main:app --host 0.0.0.0 --port 8080
    ```

=== "Start Celery worker (separate terminal)"

    ```bash
    PYTHONPATH=. celery -A app.workers.task_queue worker --loglevel=info
    ```

!!! tip "Two terminals"
    Keep the API and Celery worker running in separate terminals (or use a process manager such as `tmux`, `screen`, or `systemd` in production-like environments).

---

## 8. Verify Installation

### Check API health

```bash
curl http://localhost:8080/health
```

Expected response:

```json
{"status":"ok"}
```

### Open API documentation

- `http://localhost:8080/docs`

!!! tip "Health endpoint failed?"
    Confirm Redis is running, verify the API process is active, and inspect the API terminal logs for import/configuration errors.

---

## Manual Integrated Mode (Optional)

If you want to serve the compiled Web UI manually:

1. Extract the Web bundle ZIP
2. Set the following values in `.env`
3. Restart the API

Example `.env` additions:

```dotenv
SERVE_WEB=1
WEB_DIST_PATH=/absolute/path/to/dist
API_MOUNT_PATH=/api
WEB_API_BASE_URL=/api
```

!!! tip "Path consistency"
    Keep `API_MOUNT_PATH` and `WEB_API_BASE_URL` aligned (usually both `/api`) to avoid frontend/API routing mismatches.

---

## When to Use Manual Install

Manual installation is recommended for:

- Development environments
- CI testing
- Remote PostgreSQL servers
- Debugging migration issues
- Custom deployment environments

!!! note "Production deployments"
    For production, consider a process manager and explicit service definitions (API, Celery worker, Redis/PostgreSQL) instead of manually running long-lived commands in terminals.

---



<div style="display:flex; justify-content:space-between; align-items:center; width:100%; margin-top:2rem; gap:1rem;">
  <a href="../provision/" style="text-decoration:none; display:inline-block;">
    ← Previous: Provision
  </a>
  <a href="../upgrade/" style="text-decoration:none; display:inline-block; margin-left:auto;">
    Next: Upgrade →
  </a>
</div>
