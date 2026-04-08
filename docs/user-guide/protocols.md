---
hide:
  - toc
---

# Protocol Execution

This page describes the typical protocol flow in ScipionWeb.

## Common workflow

A standard protocol workflow usually looks like this:

1. open the relevant project
2. choose the protocol to configure
3. review required parameters and inputs
4. save configuration changes intentionally
5. launch the protocol
6. monitor execution state and logs
7. inspect outputs when execution finishes

## Before launch

Confirm these points first:

- the project context is the correct one
- required inputs are available
- parameter values were reviewed deliberately
- previous protocol state does not conflict with the next run

## During execution

Pay attention to:

- status changes
- progress indicators
- visible log output
- whether the workflow remains responsive in the browser

If the UI state does not update, refresh the project view and confirm the backend requests are succeeding.

## After execution

Once the protocol finishes, review:

- final status
- generated outputs
- whether the expected viewer or output action is available
- whether another downstream step is now unlocked

## If execution does not behave as expected

Check:

- browser network errors
- backend and worker logs
- Redis and Celery health
- whether the protocol parameters were saved correctly before launch
