# Release Checklist

Use this checklist before publishing any ScipionWeb release.

The current supported publication path is the ScipionAPI `release` command. It builds the paired API/Web artifacts by default, and `release --upload` publishes them. Routine releases should **not** require manual ZIP assembly, manual `manifest.json` editing, or manual SSH/rsync commands.

---

## Version and source state

- [ ] Release version is decided (`vX.Y.Z`)
- [ ] API and Web release versions match
- [ ] Intended release commits/tags are identified
- [ ] No accidental local/generated files are being packaged
- [ ] Release notes / known issues are prepared when needed

---

## API bundle

- [ ] Backend/unit tests are green
- [ ] Migrations apply cleanly
- [ ] CLI starts correctly
- [ ] Guided installer/provision path has been validated on a disposable machine/install
- [ ] Update works on an existing disposable installation
- [ ] Runtime folders and secrets are excluded
- [ ] `install.sh` is included in the API package
- [ ] Bundle extracts to the expected ScipionAPI layout
- [ ] Bundle name is exactly `ScipionAPI-vX.Y.Z.zip`

---

## Web bundle

- [ ] Production build succeeds
- [ ] `dist/app` contains the expected assets
- [ ] SPA routing works
- [ ] No installation-specific API URL is hardcoded
- [ ] Runtime API configuration works as expected
- [ ] Integrated API + Web behavior is validated
- [ ] Bundle name is exactly `ScipionWeb-vX.Y.Z-dist.zip`

---

## Local release build

The default `release` flow creates the paired ZIP files in `--downloads-dir`; they do not need to exist beforehand.

Before building:

- [ ] ScipionAPI declares the intended version
- [ ] ScipionWeb `package.json` declares the same version
- [ ] ScipionWeb is available as the sibling repository or through `--web-root`
- [ ] `npm` and frontend dependencies are available

Build-only command:

```bash
./scripts/scipionapi release \
  --downloads-dir /path/to/release/files
```

After building:

```text
/path/to/release/files/
├── ScipionAPI-vX.Y.Z.zip
└── ScipionWeb-vX.Y.Z-dist.zip
```

Checklist:

- [ ] `npm run build:web` completed successfully
- [ ] API ZIP exists
- [ ] Web ZIP exists
- [ ] filenames reflect the package version
- [ ] archives open/extract cleanly
- [ ] API and Web are the intended pair

!!! note "`--version` does not set package versions"
    Use `--version vX.Y.Z` only when you want the release command to assert that ScipionAPI and ScipionWeb already declare that version.

!!! note "No local `manifest.json` required"
    The release publisher downloads the current remote manifest when one exists, preserves previous releases, adds the new release, and publishes the updated manifest automatically.

!!! note "No release-directory `install.sh` required"
    The publisher reads `install.sh` from the current ScipionAPI repository root.

---

## Integration validation before publication

- [ ] Extract/inspect the API bundle
- [ ] Inspect the Web bundle
- [ ] Run a disposable integrated `provision`
- [ ] Web UI loads
- [ ] Login works
- [ ] Projects load
- [ ] Basic protocol/output path works
- [ ] `./scripts/scipionapi status` is healthy
- [ ] `./scripts/scipionapi doctor --quick` is healthy
- [ ] `./scripts/scipionapi update --dry-run` behaves correctly on a disposable existing installation
- [ ] `/api/system/version` reports the intended version after installation/update
- [ ] `/api/system/update-check` can consume the release manifest format

---

## Documentation

- [ ] Guided installation documentation is current
- [ ] Upgrade documentation is current
- [ ] CLI reference is current
- [ ] Release/publishing documentation is current
- [ ] Known issues are documented when applicable
- [ ] Support guidance points to the intended channels

---

## Real remote dry run — required

Run the publisher against the actual release target before any remote mutation. The default command rebuilds the local artifacts before evaluating the remote plan:

```bash
./scripts/scipionapi release \
  --upload \
  --downloads-dir /path/to/release/files \
  --dry-run
```

Confirm the plan reports the intended:

- [ ] version
- [ ] API ZIP
- [ ] Web ZIP
- [ ] API SHA256
- [ ] Web SHA256
- [ ] `install.sh`
- [ ] SSH login
- [ ] remote directory
- [ ] public URL
- [ ] `latest` behavior
- [ ] replacement mode (`no` for a normal new version)

The current official defaults should resolve to:

```text
SSH:    scipion@nolan.cnb.csic.es
Remote: /home/scipion/scipionfiles/downloads/scipion/scipionWeb
Public: https://scipion.cnb.csic.es/downloads/scipion/scipionWeb/
```

---

## Existing-archive compatibility path

To publish previously generated ZIP files without rebuilding them:

```bash
./scripts/scipionapi release \
  --upload \
  --no-build \
  --version vX.Y.Z \
  --downloads-dir /path/to/release/files
```

Use this path only when reusing intentionally prepared artifacts. The normal release path builds fresh archives.

---

## Existing-version protection

For a normal new release:

- [ ] `--force` is **not** required
- [ ] manifest does not already contain the target version
- [ ] target API/Web ZIP filenames do not already exist remotely

If testing the plan for an existing release, use:

```bash
./scripts/scipionapi release \
  --upload \
  --version vX.Y.Z \
  --downloads-dir /path/to/release/files \
  --force \
  --dry-run
```

!!! danger "Do not normalize release replacement"
    `--force` without `--dry-run` permits replacing an already-published version. Prefer a new patch release whenever possible.

---

## Publish

After the dry run is correct:

```bash
./scripts/scipionapi release \
  --upload \
  --downloads-dir /path/to/release/files
```

During the real publication, verify the command completes the intended order:

- [ ] ScipionAPI ZIP uploaded
- [ ] ScipionWeb ZIP uploaded
- [ ] `install.sh` published
- [ ] `manifest.json` published as the final atomic step
- [ ] public manifest verification completes or any visibility warning is understood

---

## Manifest validation after publication

The publisher creates/updates `manifest.json`; the maintainer verifies the result rather than assembling it manually.

Confirm:

- [ ] previous release entries are still present
- [ ] new `vX.Y.Z` entry exists
- [ ] API filename is correct
- [ ] Web filename is correct
- [ ] API SHA256 matches the local artifact
- [ ] Web SHA256 matches the local artifact
- [ ] file sizes look correct
- [ ] `latest` points to `vX.Y.Z` unless `--no-latest` was intentionally used

---

## Public endpoint validation

Confirm these are reachable from:

```text
https://scipion.cnb.csic.es/downloads/scipion/scipionWeb/
```

- [ ] `install.sh`
- [ ] `manifest.json`
- [ ] `ScipionAPI-vX.Y.Z.zip`
- [ ] `ScipionWeb-vX.Y.Z-dist.zip`

---

## New-user validation against the published release

Use the same public path a real user will use:

```bash
wget https://scipion.cnb.csic.es/downloads/scipion/scipionWeb/install.sh
chmod +x install.sh
./install.sh --check-only
./install.sh --version vX.Y.Z
```

Confirm:

- [ ] installer resolves the intended version
- [ ] both ZIPs download
- [ ] published SHA256 values verify
- [ ] provisioning completes
- [ ] guided installation marker is created
- [ ] selected API/Web port is persisted
- [ ] `status` / `doctor --quick` are healthy
- [ ] browser UI loads

---

## Existing-user validation against the published release

On a disposable previous-version installation:

```bash
./scripts/scipionapi update --version vX.Y.Z --dry-run
```

Then perform the controlled real update and confirm:

- [ ] update resolves the intended paired release
- [ ] checksums verify
- [ ] `SCIPION_HOME` and projects are preserved
- [ ] database migration succeeds
- [ ] Web bundle is updated
- [ ] runtime restarts successfully
- [ ] login and project loading work

---

## Final sanity questions

Before announcing/tagging the release, confirm:

- Can a new user install it using only the published `install.sh` documentation?
- Can an existing user update through `scripts/scipionapi update`?
- Is the API/Web pair reproducible and checksum-identified?
- Does the manifest still preserve older releases?
- Is the new release correctly marked as `latest` when intended?
- Did the release publisher protect against accidental replacement?
- Are documentation and support guidance ready?

---

## Tag the release

When the release state is validated according to the project's release policy:

```bash
git tag vX.Y.Z
git push origin vX.Y.Z
```

Coordinate tag timing with the project's normal release process; the public artifacts, manifest state, and source tag should describe the same release.
