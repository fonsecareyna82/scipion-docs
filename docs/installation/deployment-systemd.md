---
hide:
  - toc
---

# Production Deployment with systemd

This guide explains how to run **ScipionAPI** and the **Celery worker** as `systemd` services.

Using `systemd` is recommended for:

- Production environments
- Automatic restart on failure
- Startup on boot
- Centralized logging with `journalctl`
- Stable long-running services

!!! note "Scope"
    This page covers a **direct systemd deployment** (API + Celery worker). If you expose the service to users, a reverse proxy such as **nginx** is strongly recommended.

---

## Overview

A typical production deployment with `systemd` includes:

1. Stop CLI-managed services (if running)
2. Identify installation paths
3. Create a `systemd` service for the API
4. Create a `systemd` service for the Celery worker
5. Reload `systemd`
6. Enable services on boot
7. Start services
8. Verify status and logs

!!! tip "Prerequisite"
    Make sure the installation is already working (for example via `provision`) **before** switching to `systemd`.

---

## 1. Stop CLI-Managed Services

If you previously started services with the ScipionAPI CLI, stop them first to avoid port conflicts and duplicate workers.

```bash
./scripts/scipionapi stop
```

!!! warning "Avoid mixed process management"
    Do not run the same API/worker both through the CLI and `systemd` at the same time.

---

## 2. Locate Installation Paths

Identify the paths used by your deployment.

### Example API installation directory

```text
/opt/scipionweb/ScipionAPI-1.0.0
```

### Example `SCIPION_HOME`

```text
/opt/scipionweb/scipion_home
```

### Example Conda environment

```text
/home/youruser/miniconda3/envs/scipion4Web
```

!!! tip "Use absolute paths only"
    Always use absolute paths in `systemd` service files (`WorkingDirectory`, `ExecStart`, `SCIPION_HOME`, etc.).

---

## 3. Create the systemd Service for the API

Create the service file:

```text
/etc/systemd/system/scipionapi.service
```

### Example `scipionapi.service`

```ini
[Unit]
Description=ScipionAPI Service
After=network.target postgresql.service redis-server.service
Wants=postgresql.service redis-server.service

[Service]
Type=simple
User=youruser
WorkingDirectory=/opt/scipionweb/ScipionAPI-1.0.0
Environment=SCIPION_HOME=/opt/scipionweb/scipion_home
ExecStart=/home/youruser/miniconda3/envs/scipion4Web/bin/uvicorn app.backend.main:app --host 0.0.0.0 --port 8080
Restart=always
RestartSec=5

[Install]
WantedBy=multi-user.target
```

!!! note "Service dependencies"
    `After=` controls startup order. `Wants=` asks `systemd` to also start PostgreSQL and Redis when starting the API service.

---

## 4. Create the systemd Service for the Celery Worker

Create the service file:

```text
/etc/systemd/system/scipion-celery.service
```

### Example `scipion-celery.service`

```ini
[Unit]
Description=ScipionAPI Celery Worker
After=network.target redis-server.service
Wants=redis-server.service

[Service]
Type=simple
User=youruser
WorkingDirectory=/opt/scipionweb/ScipionAPI-1.0.0
Environment=SCIPION_HOME=/opt/scipionweb/scipion_home
ExecStart=/home/youruser/miniconda3/envs/scipion4Web/bin/celery -A app.workers.task_queue worker --loglevel=info
Restart=always
RestartSec=5

[Install]
WantedBy=multi-user.target
```

!!! tip "Celery and API separation"
    Keeping the API and Celery worker in separate services makes restarts, monitoring, and troubleshooting much easier.

---

## 5. Reload systemd

Reload the `systemd` daemon after creating or modifying service files:

```bash
sudo systemctl daemon-reload
```

---

## 6. Enable Services on Boot

Enable both services so they start automatically after reboot:

```bash
sudo systemctl enable scipionapi
sudo systemctl enable scipion-celery
```

---

## 7. Start Services

Start both services:

```bash
sudo systemctl start scipionapi
sudo systemctl start scipion-celery
```

!!! tip "Start order"
    Start order between API and Celery is usually flexible, but **Redis** must be available before the worker can process tasks.

---

## 8. Check Status

Verify both services are active:

```bash
sudo systemctl status scipionapi
sudo systemctl status scipion-celery
```

For a quick summary:

```bash
systemctl is-active scipionapi
systemctl is-active scipion-celery
```

Expected output:

```text
active
active
```

---

## Logs

Follow logs in real time with `journalctl`:

```bash
journalctl -u scipionapi -f
journalctl -u scipion-celery -f
```

!!! tip "Show recent logs only"
    Useful filters:

    - `journalctl -u scipionapi -n 100 --no-pager`
    - `journalctl -u scipion-celery -n 100 --no-pager`

---

## Optional: nginx Reverse Proxy

Using **nginx** in front of ScipionAPI is recommended for:

- HTTPS termination
- Exposing ports `80` / `443`
- Reverse proxy headers
- Load balancing (advanced setups)

### Minimal nginx proxy example

```nginx
location / {
    proxy_pass http://127.0.0.1:8080;
    proxy_set_header Host $host;
    proxy_set_header X-Real-IP $remote_addr;
    proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
    proxy_set_header X-Forwarded-Proto $scheme;
}
```

!!! warning "Integrated mode paths"
    If you use integrated mode (frontend served by the API) and custom API mount paths (for example `/api`), verify nginx forwards requests without breaking those routes.

---

## Hardening and Operational Tips (Recommended)

### Run with a dedicated service user

Instead of a personal account, consider a dedicated Linux user (for example `scipion`) to run the services.

### Restrict file permissions

Protect:

- `SCIPION_HOME/.env`
- Logs (if they may contain sensitive paths or errors)
- Project data directories

### Keep backups before upgrades

Before upgrading ScipionAPI/ScipionWeb, backup:

- PostgreSQL database
- `SCIPION_HOME/.env`
- Critical project data (if applicable)

### Test service restart behavior

After deployment, verify service restarts:

```bash
sudo systemctl restart scipionapi
sudo systemctl restart scipion-celery
sudo systemctl status scipionapi scipion-celery
```

---

## Troubleshooting

!!! warning "Service fails to start"
    Check:

    - `WorkingDirectory` exists
    - `ExecStart` points to the correct Conda environment binaries
    - `SCIPION_HOME` is correct
    - PostgreSQL and Redis are running
    - The installation works when started manually

!!! warning "`ModuleNotFoundError` in systemd logs"
    Usually caused by an incorrect `WorkingDirectory` or the wrong Python/Celery binary path in `ExecStart`.

!!! warning "Port 8080 already in use"
    Another process is already bound to the port. Stop the conflicting process or change the API port.

!!! warning "Celery starts but tasks do not run"
    Verify Redis connectivity, broker URL configuration, and check worker logs for import/configuration errors.

---

## Summary

Using `systemd` provides:

- Automatic restart on failure
- Startup on boot
- Centralized logging with `journalctl`
- Better operational stability for production deployments

A minimal production setup typically includes:

- `scipionapi.service`
- `scipion-celery.service`
- PostgreSQL
- Redis
- nginx (recommended for HTTPS/public exposure)



<div style="display:flex; justify-content:space-between; align-items:center; width:100%; margin-top:2rem; gap:1rem;">
  <a href="../upgrade/" style="text-decoration:none; display:inline-block;">
    ← Previous: Upgrade
  </a>
</div>
