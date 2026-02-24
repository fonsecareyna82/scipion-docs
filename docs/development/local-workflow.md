# Local Development Workflow

This guide describes the recommended workflow for developing ScipionAPI locally.

It assumes:

- Linux environment
- Conda installed
- PostgreSQL and Redis running locally

---

# 1. Clone Repository

```bash
git clone <repository-url>
cd ScipionAPI
```

---

# 2. Bootstrap Environment

```bash
./scripts/scipionapi bootstrap
```

Activate environment:

```bash
conda activate scipion4Web
```

---

# 3. Configure Runtime Workspace

Create a development runtime directory:

```bash
export SCIPION_HOME=$(pwd)/scipion_home
mkdir -p "$SCIPION_HOME"
```

Run install:

```bash
./scripts/scipionapi install \
  --user admin \
  --email admin@local \
  --pass changeMe
```

---

# 4. Start Development Server

Instead of using `start`, run uvicorn manually:

```bash
uvicorn app.backend.main:app --reload --host 0.0.0.0 --port 8080
```

This enables auto-reload on code changes.

---

# 5. Start Celery Worker

In another terminal:

```bash
celery -A app.workers.task_queue worker --loglevel=info
```

---

# 6. Verify

Open:

```
http://localhost:8080/docs
```

---

# Recommended Workflow

During development:

- Use manual uvicorn with `--reload`
- Do not use detached `start`
- Keep logs visible in terminal
- Use separate database for development

---

# Frontend Local Development

If developing frontend:

```
npm run dev
```

Ensure:

```
VITE_API_URL="http://localhost:8080"
```

---

# Suggested Branch Strategy

- main → stable
- devel → active development
- feature/* → feature branches