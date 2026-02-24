# Common Errors and Fixes

This section lists common development issues and how to fix them.

---

# 1. ImportError: scipion or pyworkflow not found

Cause:

- Wrong Conda environment

Fix:

```bash
conda activate scipion4Web
```

---

# 2. DATABASE_URL not set

Error:

```
KeyError: DATABASE_URL
```

Fix:

- Ensure `.env` exists
- Ensure SCIPION_HOME is exported
- Restart server

---

# 3. Alembic Migration Conflicts

Symptoms:

- Undefined table
- Duplicate revision errors
- Multiple heads

Fix:

- Inspect `alembic heads`
- Use fresh database
- Avoid editing applied migrations

---

# 4. Redis Connection Refused

Fix:

```bash
sudo systemctl start redis-server
```

---

# 5. Celery Not Processing Tasks

Check:

- Worker running
- Broker URL correct
- Redis reachable

---

# 6. CORS Errors

Symptoms:

- Browser blocks requests

Fix:

- Add correct origin in CORS middleware
- Ensure HTTPS consistency

---

# 7. JWT Invalid Signature

Cause:

- SECRET_KEY mismatch

Fix:

- Ensure same SECRET_KEY used for signing and verifying
- Clear local storage tokens

---

# 8. Port Already in Use

Error:

```
Address already in use
```

Fix:

```bash
lsof -i :8080
kill <pid>
```

---

# 9. Permission Denied in SCIPION_HOME

Fix:

```bash
chmod -R u+rw scipion_home
```

---

# General Debugging Strategy

1. Check logs
2. Verify environment variables
3. Verify Conda environment
4. Validate database connectivity
5. Restart services