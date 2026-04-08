---
hide:
  - toc
---

# Settings

The Settings area centralizes configuration that affects your account, instance behavior, or shared UI conventions.

---

## What you may find in Settings

Depending on your role and deployment, the Settings area can include:

- user preferences
- instance-level configuration views
- tag management or shared metadata helpers
- advanced configuration panels intended for administrators

---

## Before changing settings

Confirm the scope of the change:

- does it affect only your user session?
- does it affect the whole instance?
- does it affect shared project organization?
- is the change reversible if the result is not what you expected?

Settings are easier to manage safely when you understand whether they are personal, shared, or administrative.

---

## Good practice

- change one thing at a time
- verify the result immediately after saving
- avoid changing instance-level configuration unless you understand the impact
- document shared conventions when settings affect multiple users
- prefer clarity and consistency over ad-hoc changes that only one person understands

---

## Common settings mistakes

Users often run into trouble when they:

- assume a setting is personal when it is actually shared
- change multiple things before validating any of them
- forget to refresh affected views after saving
- test in the wrong environment and misread the result

---

## If a settings change does not persist

Check:

- whether the save request succeeded
- whether the page reloaded stale data
- whether your role has permission to change that section
- whether browser or API errors appeared during save
- whether another source of configuration is overriding the expected value
