# Build API Bundle

This page describes the maintainer checks for packaging ScipionAPI into the release artifact consumed by the guided installer and updater.

The canonical artifact name is:

```text
ScipionAPI-vX.Y.Z.zip
```

For example:

```text
ScipionAPI-v4.0.1.zip
```

---

## 1. Start from a clean release state

Use a reviewed release commit/tag and make sure generated runtime state is not included.

Inspect the workspace before packaging:

```bash
git status --short
```

Do not blindly delete local files just to make the tree look clean. Verify what belongs to the release and what is generated state.

---

## 2. Validate the API before packaging

At minimum confirm:

- unit/backend tests pass
- Alembic migrations are valid
- the CLI starts correctly
- `install.sh --check-only` behaves as expected on a suitable host
- `provision` works in a disposable installation
- `update` works against an existing disposable installation
- `uninstall --full --dry-run` correctly recognizes a guided test installation

Release validation should happen before upload.

---

## 3. Exclude runtime/generated artifacts

Do not package generated installation state such as:

```text
scipion_home/
.run/
.env
__pycache__/
.pytest_cache/
```

Also ensure no credentials, local database dumps, logs, temporary archives, or developer-specific configuration are accidentally included.

---

## 4. Required packaged layout

The API ZIP must extract to a valid ScipionAPI package root, directly or below one enclosing directory.

Important packaged content includes:

```text
app/
scipionapi_cli/
scripts/
alembic/
requirements.txt
pyproject.toml
alembic.ini
install.sh
README.rst
```

The guided installer validates key layout elements before copying the extracted API into the final installation directory.

!!! important "Include `install.sh`"
    The root installer is a managed ScipionAPI file. Keeping it in the API package allows installed systems to receive updated installer logic through the normal updater.

---

## 5. Create the release archive

The exact packaging mechanism can be scripted or performed by CI, but the final file must use the canonical name:

```text
ScipionAPI-vX.Y.Z.zip
```

Example generic ZIP command from a prepared staging directory:

```bash
zip -r "ScipionAPI-v4.0.1.zip" <prepared-api-root>/
```

Do not introduce architecture suffixes such as `-linux-x86_64` unless the release format is intentionally changed everywhere. The current installer/updater/publisher convention is `ScipionAPI-vX.Y.Z.zip`.

---

## 6. Inspect the archive

Before publication:

```bash
unzip -l ScipionAPI-v4.0.1.zip | less
```

Confirm:

- required source/config files are present
- `scripts/scipionapi` is present
- `install.sh` is present
- Alembic files are present
- runtime data is absent
- secrets are absent

Then extract the ZIP into a temporary directory and run a disposable installation/provision validation.

---

## 7. Do not upload the API ZIP manually

The supported publication flow uploads the API and Web pair together:

```bash
./scripts/scipionapi release \
  --upload \
  --version v4.0.1 \
  --downloads-dir /path/to/release/files \
  --dry-run
```

Then, after reviewing the plan:

```bash
./scripts/scipionapi release \
  --upload \
  --version v4.0.1 \
  --downloads-dir /path/to/release/files
```

The publisher calculates SHA256 metadata and updates `manifest.json` automatically. A separate `.sha256` sidecar file is not required by the current release flow.

---

## API bundle checklist

- [ ] release version is correct
- [ ] tests are green
- [ ] migrations are valid
- [ ] packaged filename is `ScipionAPI-vX.Y.Z.zip`
- [ ] expected ScipionAPI layout is present
- [ ] `install.sh` is present
- [ ] runtime/generated directories are absent
- [ ] no secrets/logs/local state are included
- [ ] archive extracts cleanly
- [ ] disposable provision/install validation succeeds
- [ ] matching Web artifact exists before publication

Continue with [Build Web Bundle](build-web-bundle.md), then [Publish a ScipionWeb Release](publishing.md).
