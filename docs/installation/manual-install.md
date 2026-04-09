---
hide:
  - toc
---

# Manual Installation

This guide describes how to install **ScipionAPI** manually, step by step.

Use this method if:

- you prefer full control over the installation
- you are using a remote PostgreSQL server
- you do not want automatic database bootstrap
- you are debugging or developing locally

!!! note "Manual vs one-shot provisioning"
    If you want the fastest setup path, use [`provision`](../provision/). This page is intended for **controlled, step-by-step installation**.

---

## Mental model

Manual installation is the right path when you want to separate these stages clearly:

1. Python environment
2. database and runtime configuration
3. migrations and admin creation
4. service startup and verification

That separation is slower than `provision`, but much better for debugging and advanced deployments.

---

## Overview

Manual installation typically consists of:

1. creating a Conda environment
2. installing Python dependencies
3. configuring PostgreSQL manually
4. running Alembic migrations
5. creating the admin user
6. starting API and Celery services

---

## 1. Create Conda Environment

Run the following commands from inside the extracted **ScipionAPI** directory:

```bash
conda create -n scipion4Web python=3.8 -y
conda activate scipion4Web
python -m pip install --upgrade pip
```

---

## 2. Install Dependencies

```bash
pip install -r requirements.txt
pip install -e .
scipionapi --help
```

!!! tip "Editable install"
    `pip install -e .` is recommended for local development because code changes are picked up without reinstalling the package.

---

## 3. Create PostgreSQL Database Manually

If you are not using automatic database bootstrap, create the PostgreSQL role and database manually.

Typical outline:

- create role
- create database
- grant privileges
- verify connectivity

The key point is to be sure the values you create here match what you will later place in `.env`.

---

## 4. Configure Environment (`.env`)

Create a runtime workspace such as:

```bash
mkdir -p scipion_home
export SCIPION_HOME="$(pwd)/scipion_home"
```

Then create:

```text
scipion_home/.env
```

Populate it with a valid `DATABASE_URL`, `SECRET_KEY`, API host and port, broker URL, and runtime paths.

---

## 5. Run Alembic Migrations

Load the environment values and apply migrations:

```bash
set -a
source scipion_home/.env
set +a
alembic upgrade head
```

At this point the database schema should be ready.

---

## 6. Create Admin User

Run:

```bash
scipionapi install \
  --user "admin" \
  --email "admin@example.com" \
  --pass "changeMe"
```

This step ensures the admin user exists and aligns runtime configuration with the installation state.

---

## 7. Start Services

Start the API and the Celery worker, then verify:

```bash
curl http://localhost:8080/health
```

If you are running manually, keep API and worker logs visible in separate terminals while testing.

---

## When this path is best

Manual installation is recommended for:

- development environments
- CI testing
- remote PostgreSQL servers
- debugging migration issues
- custom deployment environments

---

## Common mistakes

!!! warning "Environment prepared, but `.env` still wrong"
    A healthy Python environment does not guarantee a healthy runtime configuration.

!!! warning "Database exists, but migration state is unclear"
    When in doubt, test the migration flow on a fresh empty database.

!!! warning "Services start, but worker side is forgotten"
    Always verify both the API and Celery sides of the runtime.
