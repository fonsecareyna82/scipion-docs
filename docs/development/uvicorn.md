# Running Uvicorn Manually

For development, it is recommended to run Uvicorn manually instead of using the CLI `start` command.

---

# Basic Command

```bash
uvicorn app.backend.main:app --reload --host 0.0.0.0 --port 8080
```

---

# Why Use --reload?

- Automatically reloads on file changes
- Faster development cycle
- No need to restart manually

---

# Required Environment

Before running:

```bash
conda activate scipion4Web
export SCIPION_HOME=$(pwd)/scipion_home
```

Ensure `.env` exists inside `SCIPION_HOME`.

---

# Running on Different Port

```bash
uvicorn app.backend.main:app --reload --port 9000
```

---

# Production Warning

Do NOT use:

```
--reload
```

In production.

Production should use:

- systemd
- gunicorn + uvicorn workers
- reverse proxy

---

# Example Production Command

```
uvicorn app.backend.main:app --host 0.0.0.0 --port 8080
```

---

# Troubleshooting

If import errors appear:

- Verify correct Conda environment
- Verify PYTHONPATH includes project root
- Verify SCIPION_HOME is set