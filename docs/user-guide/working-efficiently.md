---
hide:
  - toc
---

# Working Efficiently

This page collects practical habits that make day-to-day work in ScipionWeb more reliable and easier to understand.

---

## 1. Confirm context before acting

Before launching, sharing, renaming, or reviewing results, verify:

- the environment you are using
- the project you are in
- the protocol or output you are looking at
- whether the visible state belongs to the latest workflow step

A large fraction of user confusion comes from acting in the wrong context rather than from a broken feature.

---

## 2. Prefer clear naming over fast naming

Use names that help you and your teammates answer basic questions quickly:

- what is this project about?
- which dataset or experiment does it represent?
- is this the current workflow or an older attempt?

Good names reduce mistakes later in protocol execution, sharing, and output review.

---

## 3. Treat save and launch as different moments

A good mental model is:

- configure carefully
- save intentionally
- launch only when the saved state is the one you want

This helps avoid confusion about whether a protocol was merely edited or actually executed.

---

## 4. Review outputs early

Do not wait too long to inspect important outputs.

Early review helps you detect:

- incorrect input selection
- unexpected parameter choices
- runs that completed but produced the wrong kind of result
- downstream steps that should not continue yet

---

## 5. Coordinate shared work explicitly

In collaborative environments:

- communicate before high-impact project actions
- make it clear which project state is current
- avoid silent assumptions about who validated which outputs
- keep naming and ownership conventions stable

---

## 6. Use the browser as a debugging surface

When something feels inconsistent, inspect:

- browser network requests
- failed API responses
- visible route or reload behavior
- whether the page is showing stale state

You do not need to be a backend developer to benefit from checking the browser first.

---

## 7. Recheck before assuming the system is wrong

Before concluding that something failed, quickly verify:

- session is still valid
- the correct environment is open
- the right project is selected
- the expected protocol run is visible
- the output belongs to the workflow step you think it does

This habit saves a surprising amount of time.
