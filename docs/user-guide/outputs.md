---
hide:
  - toc
---

# Outputs and Viewers

This page covers the basic expectations when inspecting results in ScipionWeb.

## Typical output workflow

After a protocol finishes, you will usually:

- locate the generated outputs
- open the relevant viewer or preview
- inspect the data in the context of the project and protocol history
- decide whether the workflow can continue with confidence

## What to check when opening outputs

Before assuming a viewer is broken, confirm that:

- the protocol completed successfully
- the selected output belongs to the expected run
- the viewer matches the output type
- browser and backend requests are succeeding

## While inspecting results

Review outputs with these questions in mind:

- is this the expected result for the current workflow step?
- does the output belong to the intended protocol execution?
- are there obvious signs that the run failed or produced incomplete data?
- do follow-up protocols now have the inputs they need?

## If a viewer does not open

Check the following first:

- failed browser network requests
- missing backend viewer data
- stale frontend state after route changes
- incomplete protocol execution

## Good practice

Inspect outputs in the context of the project and protocol history, not as isolated files.
