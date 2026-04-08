---
hide:
  - toc
---

# First Login and Session Basics

Use this page for the first actions after opening ScipionWeb in the browser.

---

## Before you troubleshoot login

Confirm these basics first:

- you are opening the intended environment
- the backend instance is actually reachable
- your browser is not reusing stale session state from another instance
- you are not mixing tabs from different environments

---

## After login, verify that

- the main navigation loads correctly
- your user identity appears as expected
- the projects area opens without errors
- requests complete without browser auth or CORS failures
- protected data loads consistently after refresh

If the interface appears but project-related data never loads, the problem is often session state or backend/API connectivity rather than visual rendering.

---

## Common first-login problems

### Invalid credentials

Check that you are using the correct username, password, and environment.

### Login works but protected pages fail

Possible causes include:

- stale token stored in the browser
- backend `SECRET_KEY` changed
- API base URL mismatch
- session state was created against another environment

### The page loads but actions fail

Open browser DevTools and inspect failed network requests under `/api`.

---

## Useful recovery actions

Try these steps in order:

1. log out and log in again
2. refresh the page after a clean sign-in
3. clear stale browser storage if the instance configuration changed recently
4. retry from a private browser window to isolate local-state issues

!!! tip "Private window test"
    A private window is often the fastest way to check whether the issue is caused by cached tokens or stale local browser state.

---

## Good habits for multi-environment work

If you regularly switch between local, staging, and production:

- keep those environments in clearly named bookmarks
- avoid leaving old tabs open for long periods
- confirm the URL before changing protected data
- do not assume a saved session belongs to the environment you intend to use now

---

## When to ask an administrator for help

Ask for help when:

- the same issue reproduces across browsers or machines
- your account should exist but access is denied
- the instance is reachable but protected data never loads
- the backend appears healthy but login still fails consistently
