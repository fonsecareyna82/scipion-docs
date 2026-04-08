---
hide:
  - toc
---

# Outputs and Viewers

This page covers the basic expectations when inspecting results in ScipionWeb.

## Typical output workflow

After a protocol finishes, you will usually:

- locate the generated outputs
- open the relevant viewer
- inspect the data or preview
- compare results with the expected workflow stage

## What to check when opening outputs

- the protocol completed successfully
- the selected output belongs to the expected run
- the viewer matches the output type
- browser and backend requests succeed

## If a viewer does not open

Check the following first:

- failed browser network requests
- missing backend viewer data
- stale frontend state after route changes
- incomplete protocol execution

## Good practice

Inspect outputs in the context of the project and protocol history, not as isolated files.
