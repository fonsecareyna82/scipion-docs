# Protocol Execution

This page describes the typical protocol flow in ScipionWeb: configure parameters, save intentionally, launch execution, monitor progress, and inspect outputs.

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

## Open and configure a protocol

1. Open the intended project.
2. Locate the protocol in the workflow view.
3. Open the protocol form or details panel.
4. Review required parameters first.
5. Select inputs deliberately, especially pointer-like parameters that refer to outputs from previous protocols.
6. Review optional parameters only after the required inputs are correct.

Before changing values, confirm that you are editing the intended protocol run and not a similarly named one.

---

## Save before launching

If the form separates saving from execution, treat **Save** and **Execute** as different actions:

1. Change parameter values.
2. Save the protocol configuration.
3. Confirm that validation errors, if any, are resolved.
4. Launch only after the saved values represent the run you want.

!!! warning "Saved configuration vs execution"
    A protocol can be configured without being launched. Likewise, visible outputs may belong to a previous execution if a new launch has not finished yet.

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

## Launch and monitor execution

1. Start execution from the protocol action or form.
2. Watch the protocol status.
3. Open logs when available.
4. Keep an eye on progress indicators and error messages.
5. Refresh the project view if status appears stale.

During execution, pay attention to:

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

A good habit is to inspect outputs before launching dependent downstream protocols.

---

## Common sources of confusion

Users often confuse these states:

- parameters changed but not saved
- configuration saved but launch not triggered yet
- execution started but the worker has not processed the task yet
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

Useful checks for administrators or developers:

```bash
./scripts/scipionapi status
./scripts/scipionapi logs
redis-cli ping
```

If tasks remain pending, see [Known Issues and Workarounds](../support/known-issues/).
