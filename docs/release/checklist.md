# Release Checklist

Use this checklist before publishing any ScipionWeb release.

---

## API Bundle

- [ ] Clean working directory
- [ ] Migrations apply cleanly
- [ ] Provision works on fresh machine
- [ ] No secrets in repo
- [ ] No runtime folders included
- [ ] README updated
- [ ] Version bumped

---

## Web Bundle

- [ ] Build succeeds
- [ ] dist contains correct assets
- [ ] SPA routing works
- [ ] No hardcoded API URLs
- [ ] config.js injected correctly

---

## Integration Test

- [ ] Download API bundle
- [ ] Download Web bundle
- [ ] Extract both
- [ ] Run provision with --web-dist
- [ ] Access UI in browser
- [ ] Login works
- [ ] Projects load

---

## Security

- [ ] SECRET_KEY generated per install
- [ ] No credentials in bundles
- [ ] HTTPS documented

---

## Performance

- [ ] API startup reasonable
- [ ] Celery worker starts clean
- [ ] No blocking tasks

---

## Documentation

- [ ] Install guide updated
- [ ] Upgrade guide updated
- [ ] Known issues documented

---

## Release Artifacts

- [ ] API zip uploaded
- [ ] Web zip uploaded
- [ ] Checksums uploaded (optional)
- [ ] Version announcement prepared

---

# Final Step

Tag release:

```bash
git tag v1.2.0
git push origin v1.2.0
```

---

# Release Complete 🎉