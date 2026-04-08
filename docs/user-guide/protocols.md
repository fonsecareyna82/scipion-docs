---
hide:
  - toc
---

# Protocol Execution

This page describes the typical protocol flow in ScipionWeb.

---

## Common workflow

A standard protocol workflow usually looks like this:

1. open the relevant project
2. choose the protocol to configure
3. review required parameters and inputs
4. save configuration changes intentionally
5. launch the protocol
6. monitor execution state and logs
7. inspect outputs when execution finishes

---

## Before launch

Confirm these points first:

- the project context is the correct one
- required inputs are available
- parameter values were reviewed deliberately
- previous protocol state does not conflict with the next run
- you understand whether you are configuring, saving, or launching

A careful pre-launch check is often the difference between a clean workflow and a confusing rerun later.

---

## During execution

Pay attention to:

- status changes
- progress indicators
- visible log output
- whether the workflow remains responsive in the browser

If the UI state does not update, refresh the project view and confirm the backend requests are succeeding.

---

## After execution

Once the protocol finishes, review:

- final status
- generated outputs
- whether the expected viewer or output action is available
- whether another downstream step is now unlocked
- whether the result matches what the workflow stage should have produced

---

## Common sources of confusion

Users often confuse these states:

- parameters changed but not saved
- configuration saved but launch not triggered yet
- execution finished but outputs not reviewed yet
- visible outputs belong to an earlier run rather than the latest one

Whenever something feels inconsistent, go back to the project context and verify the full protocol lifecycle again.

---

## If execution does not behave as expected

Check:

- browser network errors
- backend and worker logs
- Redis and Celery health
- whether the protocol parameters were saved correctly before launch
- whether you are looking at the correct protocol run and project state
