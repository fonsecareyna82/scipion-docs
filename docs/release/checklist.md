# Release Checklist

Use this checklist before publishing any ScipionWeb release.

---

## API Bundle

- [ ] Clean working directory
- [ ] Migrations apply cleanly
- [ ] Provision works on a fresh machine
- [ ] No runtime folders included
- [ ] README updated
- [ ] Version bumped

---

## Web Bundle

- [ ] Build succeeds
- [ ] `dist` contains the expected assets
- [ ] SPA routing works
- [ ] No hardcoded API URLs
- [ ] runtime config injection works as expected

---

## Integration Validation

- [ ] Download API bundle
- [ ] Download Web bundle
- [ ] Extract both
- [ ] Run `provision` with `--web-dist`
- [ ] Access UI in browser
- [ ] Login works
- [ ] Projects load

---

## Documentation

- [ ] Install guide updated
- [ ] Upgrade guide updated
- [ ] Known issues documented
- [ ] Support guidance still points to the correct public channels

---

## Release Artifacts

- [ ] API zip uploaded
- [ ] Web zip uploaded
- [ ] version announcement prepared

---

## Final Sanity Questions

Before tagging, confirm:

- would a new user be able to install this version from the published docs?
- would an existing user understand the upgrade path?
- are the support channels ready for incoming questions or bug reports?

---

## Final Step

Tag the release:

```bash
git tag v1.2.0
git push origin v1.2.0
```
