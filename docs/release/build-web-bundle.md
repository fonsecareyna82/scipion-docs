# Build Web Bundle

This page describes how to build and package the ScipionWeb frontend for a paired release.

The canonical Web artifact name is:

```text
ScipionWeb-vX.Y.Z-dist.zip
```

For example:

```text
ScipionWeb-v4.0.1-dist.zip
```

---

## 1. Install frontend dependencies

From the ScipionWeb source tree:

```bash
npm install
```

Use the repository's normal lockfile/package-manager policy when preparing a production release.

---

## 2. Build the production frontend

The release builder invokes the production Web build as:

```bash
npm run build:web
```

The release flow expects the compiled application at:

```text
dist/app/
```

Running `./scripts/scipionapi release` from ScipionAPI performs this Web build automatically.

---

## 3. Validate the build

Confirm the expected production files exist, including:

```text
dist/app/index.html
dist/app/assets/
```

Also validate the behavior relevant to ScipionWeb:

- SPA routes load correctly
- static assets resolve correctly
- no installation-specific API URL is hardcoded into the build
- runtime API configuration can be injected/deployed by ScipionAPI
- the build works with the matching ScipionAPI release

A simple static server can help inspect the standalone build:

```bash
npx serve dist/app
```

This does not replace integrated API/Web validation.

---

## 4. Runtime configuration principle

The same compiled Web bundle should be reusable across installations.

ScipionAPI deploys runtime configuration for the frontend rather than requiring a rebuild for every host/port.

Conceptually, the deployed configuration tells the frontend which API base path/URL to use, commonly `/api` in integrated mode.

This provides:

- one Web artifact per release
- no host-specific rebuild
- easier caching/distribution
- clean integrated and separate deployment models

---

## 5. Package the Web release

The supported release path packages the Web archive automatically:

```bash
./scripts/scipionapi release \
  --downloads-dir /path/to/release/files
```

The builder reads `dist/app/` and stores its contents under an `app/` root in the ZIP. For example:

```text
app/
├── index.html
└── assets/
```

The final filename follows:

```text
ScipionWeb-vX.Y.Z-dist.zip
```

Manual ZIP creation can still be useful for debugging packaging itself, but the normal release should use `scipionapi release` so its layout matches the installer/updater expectations.

Do not use the old lowercase convention (`scipionweb-...`) in new releases; the installer/updater/publisher use the canonical `ScipionWeb-...` naming.

---

## 6. Inspect the archive

```bash
unzip -l ScipionWeb-v4.0.1-dist.zip | less
```

Confirm that the release archive contains the expected packaged application root:

```text
app/index.html
app/assets/
```

Then validate the archive with the matching ScipionAPI bundle in a disposable integrated installation.

---

## 7. Do not upload the Web ZIP manually

The normal publisher rebuilds both artifacts before publication. Run:

```bash
./scripts/scipionapi release \
  --upload \
  --downloads-dir /path/to/release/files \
  --dry-run
```

After reviewing the plan:

```bash
./scripts/scipionapi release \
  --upload \
  --downloads-dir /path/to/release/files
```

The publisher handles checksums, remote validation, `install.sh`, and `manifest.json`.

---

## Web bundle checklist

- [ ] production build succeeds
- [ ] `dist/app/index.html` exists
- [ ] static assets are present
- [ ] SPA routing works
- [ ] no installation-specific API URL is hardcoded
- [ ] runtime API configuration works
- [ ] filename is `ScipionWeb-vX.Y.Z-dist.zip`
- [ ] archive contains a valid `app/` root
- [ ] matching API artifact exists
- [ ] integrated disposable-install validation succeeds

Continue with [Publish a ScipionWeb Release](publishing.md).
