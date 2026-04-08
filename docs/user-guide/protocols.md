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
3. review required parameters
4. save configuration changes
5. launch the protocol
6. monitor execution status
7. inspect outputs when execution finishes

## What to verify before launch

- required inputs are available
- parameter values are intentional
- the project context is the correct one
- previous protocol state does not conflict with the next execution

## Monitoring execution

During execution, pay attention to:

- status changes
- progress indicators
- log visibility
- output availability after completion

## If execution does not behave as expected

Check:

- browser network errors
- backend and worker logs
- Redis and Celery health
- whether the protocol parameters were saved correctly before launch
