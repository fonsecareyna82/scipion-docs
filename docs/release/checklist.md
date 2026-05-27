# Release Checklist

Use this checklist before publishing any ScipionWeb release.

---

## API Bundle

- [ ] Clean working directory
- [ ] Migrations apply cleanly
- [ ] Provision works on a fresh machine
- [ ] `update` works on an existing test installation
- [ ] No runtime folders included
- [ ] README updated
- [ ] Version bumped
- [ ] Bundle name follows the public convention: `ScipionAPI-vX.Y.Z.zip`

---

## Web Bundle

- [ ] Build succeeds
- [ ] `dist` contains the expected assets
- [ ] SPA routing works
- [ ] No hardcoded API URLs
- [ ] Runtime config injection works as expected
- [ ] Bundle name follows the public convention: `ScipionWeb-vX.Y.Z-dist.zip`

---

## Integration Validation

- [ ] Download API bundle
- [ ] Download Web bundle
- [ ] Extract both
- [ ] Run `provision` with `--web-dist`
- [ ] Access UI in browser
- [ ] Login works
- [ ] Projects load
- [ ] Run `./scripts/scipionapi update --dry-run`
- [ ] Run `./scripts/scipionapi update --version vX.Y.Z --force` in a disposable test install
- [ ] Confirm `/api/system/version` reports the expected installed version
- [ ] Confirm `/api/system/update-check` can read the public manifest

---

## Documentation

- [ ] Install guide updated
- [ ] Upgrade guide updated
- [ ] CLI `update` reference updated
- [ ] Known issues documented
- [ ] Support guidance still points to the correct public channels

---

## Release Artifacts

- [ ] API ZIP created: `ScipionAPI-vX.Y.Z.zip`
- [ ] Web ZIP created: `ScipionWeb-vX.Y.Z-dist.zip`
- [ ] Existing `manifest.json` downloaded or copied locally
- [ ] `manifest.json` regenerated with the new release entry
- [ ] Previous release entries are still present in `manifest.json`
- [ ] `latest` points to the new version
- [ ] SHA256 values generated for both ZIP files
- [ ] API ZIP uploaded
- [ ] Web ZIP uploaded
- [ ] `manifest.json` uploaded last
- [ ] Direct URLs for both ZIP files return HTTP 200
- [ ] Direct URL for `manifest.json` returns HTTP 200
- [ ] Version announcement prepared

Generate or update the manifest with:

```bash
python scripts/update_release_manifest.py \
  --version vX.Y.Z \
  --downloads-dir /path/to/releases
```

The release directory should include the current manifest before running the script:

```text
manifest.json
ScipionAPI-vX.Y.Z.zip
ScipionWeb-vX.Y.Z-dist.zip
```

Upload `manifest.json` only after both ZIP files are available on the server.

---

## Final Sanity Questions

Before tagging, confirm:

- Would a new user be able to install this version from the published docs?
- Would an existing user understand the update path?
- Does `./scripts/scipionapi update --dry-run` show the expected target version?
- Does the Home dashboard correctly report update availability?
- Are the support channels ready for incoming questions or bug reports?

---

## Final Step

Tag the release:

```bash
git tag vX.Y.Z
git push origin vX.Y.Z
```
