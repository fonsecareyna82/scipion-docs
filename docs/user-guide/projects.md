---
hide:
  - toc
---

# Projects

Projects are the main workspace unit in ScipionWeb. They usually group protocols, outputs, logs, and related analysis context.

---

## Typical project workflow

A common project-oriented flow looks like this:

1. create or open the intended project
2. confirm the project identity and scope
3. review visible protocols and outputs
4. perform the next workflow action from the correct project context
5. share the project if collaboration is needed

---

## When creating a project

Use a name that is:

- short enough to scan quickly
- descriptive enough to distinguish it from similar work
- stable enough to avoid repeated renames later

If your team follows a naming convention, apply it from the beginning.

---

## When opening a project

Verify these basics first:

- project name and description match the expected workflow
- visible protocols correspond to the intended dataset or experiment
- current status and recent outputs look reasonable
- you are not accidentally working in a similarly named project

This matters because many later actions depend on being in the right project context before launching, reviewing, or sharing anything.

---

## Renaming and lifecycle actions

Use rename when the project remains relevant but its label needs clarification.

Before high-impact project actions, confirm:

- no teammate is actively depending on the same project state
- critical outputs were already reviewed or backed up
- the action is being applied in the intended environment
- you understand whether the project is private or shared

!!! warning "Shared work needs coordination"
    In collaborative environments, project-level actions should be treated as workflow changes, not just cosmetic edits.

---

## Project hygiene recommendations

- keep names consistent
- avoid near-duplicate project labels
- review old projects deliberately instead of letting them accumulate without context
- document collaboration expectations when the same project is shared by several users
- prefer clarity over short-lived convenience in naming and organization

---

## If a project action fails

Check:

- whether you are still authenticated
- whether the project list refreshes correctly
- whether browser requests are failing with auth or CORS errors
- whether the backend API returns validation or permission errors
- whether you are looking at stale UI state after a route change
