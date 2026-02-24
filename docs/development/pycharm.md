# Debugging with PyCharm

This guide explains how to configure PyCharm for backend debugging.

---

# 1. Open Project

Open the ScipionAPI directory in PyCharm.

---

# 2. Configure Python Interpreter

Go to:

```
Settings → Project → Python Interpreter
```

Select:

```
Conda environment → scipion4Web
```

If not detected, manually point to:

```
~/miniconda3/envs/scipion4Web/bin/python
```

---

# 3. Create Run Configuration

Create a new Python configuration:

- Script path: leave empty
- Module name:

```
uvicorn
```

Parameters:

```
app.backend.main:app --reload --host 0.0.0.0 --port 8080
```

Working directory:

```
<project-root>
```

Environment variables:

```
SCIPION_HOME=/absolute/path/to/scipion_home
```

---

# 4. Enable Debugging

Set breakpoints inside:

- routers
- services
- models
- auth dependencies

Run in Debug mode.

---

# 5. Debug Celery Tasks

For Celery debugging:

Instead of running separate worker, temporarily call task synchronously in code.

Or configure Celery to run in eager mode (development only).

---

# Common Debug Tips

- Ensure correct Conda interpreter selected
- Ensure SCIPION_HOME is set
- Verify `.env` is loaded
- Use PyCharm database tool to inspect PostgreSQL

---

# Recommended PyCharm Settings

- Enable "Reload changed modules"
- Enable "Show console"
- Disable aggressive auto-import if interfering