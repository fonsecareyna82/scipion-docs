# Build API Bundle

This document explains how to package the ScipionAPI backend into a distributable release bundle.

---

# Step 1 — Clean Workspace

From repo root:

```bash
git clean -xfd
```

Or manually ensure:

- No logs
- No .run/
- No scipion_home/
- No venvs

---

# Step 2 — Validate Production State

Ensure:

- Migrations pass
- Tests pass (if present)
- Provision works on clean machine

---

# Step 3 — Exclude Runtime Artifacts

Make sure these are NOT included:

```
scipion_home/
.run/
.env
__pycache__/
```

---

# Step 4 — Create Release Folder

```bash
mkdir scipionapi-1.2.0
rsync -av --exclude scipion_home --exclude .run ./ scipionapi-1.2.0/
```

---

# Step 5 — Create ZIP

```bash
zip -r scipionapi-1.2.0-linux-x86_64.zip scipionapi-1.2.0/
```

---

# Step 6 — Upload

Upload bundle to release server:

```
https://scipion.cnb.csic.es/downloads/scipion/scipionWeb/
```

---

# Optional: Checksum

```bash
sha256sum scipionapi-1.2.0-linux-x86_64.zip > scipionapi-1.2.0.sha256
```

---

# Recommended Automation

Eventually automate with:

- Makefile
- GitHub Actions
- CI pipeline

---

# Validation Checklist

- Bundle extracts cleanly
- README included
- Provision works from extracted folder
- No secrets present