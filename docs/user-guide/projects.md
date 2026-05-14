# Projects

Projects are the main workspace unit in ScipionWeb. They group protocols, outputs, logs, and related analysis context.

Use this page when you need to create, open, rename, review, or share a project safely.

---

## Typical project workflow

A common project-oriented flow looks like this:

1. open the **Projects** page
2. create or open the intended project
3. confirm the project identity and scope
4. review visible protocols and outputs
5. perform the next workflow action from the correct project context
6. share the project if collaboration is needed

---

## Create a project

1. Open the **Projects** page.
2. Select **New Project**.
3. Enter a short but descriptive project name.
4. Add a description if your team uses descriptions to document project purpose.
5. Confirm the creation action.
6. Open the new project and verify that the project page loads correctly.

After creating a project, confirm:

- the project appears in the project list
- the name is easy to distinguish from similar projects
- the project opens without API or authentication errors

---

## Open an existing project

1. Open the **Projects** page.
2. Locate the intended project card or list entry.
3. Check the project name, description, thumbnail, or recent status before opening it.
4. Open the project.
5. Review the visible workflow before launching or editing anything.

Verify these basics first:

- project name and description match the expected workflow
- visible protocols correspond to the intended dataset or experiment
- current status and recent outputs look reasonable
- you are not accidentally working in a similarly named project

This matters because many later actions depend on being in the right project context before launching, reviewing, or sharing anything.

---

## Rename a project

Use rename when the project remains relevant but its label needs clarification.

1. Open the project actions menu.
2. Select the rename action.
3. Enter the new project name.
4. Confirm the change.
5. Refresh or revisit the project list if the old name still appears temporarily.

Before renaming, confirm:

- no teammate is actively depending on the previous name
- the new name follows your team convention
- the rename is being applied in the intended environment

---

## Share a project

1. Open the project actions menu.
2. Select the share action.
3. Choose the target user or collaborator according to the available sharing UI.
4. Confirm the sharing action.
5. Ask the collaborator to refresh their project list if the project does not appear immediately.

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

Useful recovery steps:

1. refresh the project list
2. log out and log in again if authentication looks stale
3. inspect failed browser network requests
4. check API logs if the backend returns an error
