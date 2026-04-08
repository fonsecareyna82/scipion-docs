---
hide:
  - toc
---

# First Login and Session Basics

Use this page for the first actions after opening ScipionWeb in the browser.

## Before you troubleshoot login

Confirm these basics first:

- you are opening the intended environment
- the backend instance is actually reachable
- your browser is not reusing stale session state from another instance

## After login, verify that

- the main navigation loads correctly
- your user identity appears as expected
- the projects area opens without errors
- requests complete without browser auth or CORS failures

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

## Useful recovery actions

- log out and log in again
- refresh the page after a clean sign-in
- clear stale browser storage if the instance configuration changed recently
- retry from a private browser window to isolate local-state issues

## When to ask an administrator for help

Ask for help when:

- the same issue reproduces across browsers or machines
- your account should exist but access is denied
- the instance is reachable but protected data never loads
- the backend appears healthy but login still fails consistently
