# Settings

The **Settings** area centralizes configuration that affects how ScipionWeb behaves for users, projects, plugins, protocol execution, and runtime integration with external systems.

Use this page as the entry point for understanding what can be configured from the web interface and which dedicated guide to follow for each settings area.

<img src="../../assets/images/screenshots/user-guide/settings-page.png" alt="Settings page" style="display:block;width:100%;max-width:980px;height:auto;margin:1.2rem auto 1.6rem;" />

*Settings page with user, instance, tags, environment, and host-related configuration areas.*

---

## What Settings is for

Settings is not only a place for personal preferences. In ScipionWeb, it can also expose shared configuration that affects project organization, plugin integration, and how protocols are submitted for execution.

Depending on your role and deployment, Settings may include:

- user preferences
- instance-level configuration
- project tag management
- environment variables for plugins and external software
- host and queue configuration for protocol execution
- advanced configuration views for administrators

!!! warning "Some settings affect all users"
    Before changing instance, environment, host, or queue settings, confirm whether the change affects only your account or the whole ScipionWeb deployment.

---

## Main settings areas

### User settings

User settings usually affect your own experience in ScipionWeb.

They may include preferences such as workflow display behavior, UI options, refresh intervals, or other per-user defaults.

Change user settings when you want to adapt the interface to how you work, without changing how protocols run for everyone else.

---

### Instance settings

Instance settings affect the ScipionWeb deployment as a whole.

They may control runtime behavior, global defaults, or administrative options that should remain consistent across users.

Only change instance settings when you understand the deployment impact and can validate the result afterward.

---

### Tags

Tags help users organize protocols visually inside a project. They can be used to mark workflow stages, review status, candidate results, deprecated branches, or any team-specific classification.

Use the dedicated guide when you need to create, edit, delete, or assign tags:

[Open the Tags guide](tags/)

---

### Environment variables

Environment variables managed from Settings are useful when ScipionWeb needs to expose runtime values to plugins or external software.

A common case is when an external program is already installed on the server, and the plugin only needs a path variable such as `PLUGIN_HOME` pointing to that installation.

Use the dedicated guide when you need to add or modify runtime variables from the web interface:

[Open the Environment Variables guide](environment-variables/)

---

### Hosts and queues

Host settings define how ScipionWeb submits protocol executions, checks job status, and cancels jobs when a queue system is used.

This is where SLURM-related queue execution can be configured from the web interface, including submit commands, cancel commands, check commands, submit templates, queues, memory, time, CPU, and GPU parameters.

Use the dedicated guide when you need to configure protocol execution through SLURM or another queue system:

[Open the Hosts and Queues guide](hosts-and-queues/)

---

### Advanced configuration

Some deployments may expose advanced or read-only configuration views.

Use these sections carefully. They are useful for reviewing the current state, debugging configuration issues, or confirming what ScipionWeb has loaded, but they may not be intended for routine editing by all users.

---

## Recommended workflow before changing settings

Before changing anything, confirm:

1. which area you are changing
2. whether the change is personal, project-level, runtime-level, queue-level, or instance-wide
3. whether other users may be affected
4. how you will validate the change
5. how to revert the change if the result is not correct

This is especially important for environment variables and host/queue settings, because they can affect plugin discovery and protocol execution.

---

## Validation after changes

After saving settings, validate the smallest possible workflow that proves the change worked.

Examples:

- after changing a user preference, refresh the affected view
- after editing tags, open a project and confirm that tag assignment still works
- after changing an environment variable, open or run a plugin protocol that depends on it
- after changing host or queue settings, launch a small protocol through the configured queue
- after changing instance settings, verify that the expected behavior is visible for the intended users

---

## Good practice

- change one setting at a time
- save and validate before making another change
- document shared conventions for tags, environment variables, and queues
- avoid changing host or queue settings while protocols are being launched or debugged
- avoid changing runtime paths without validating the affected plugin
- keep queue names aligned with the real scheduler configuration
- prefer clear names and conservative defaults over ad-hoc values

---

## Common settings mistakes

Users often run into trouble when they:

- assume a setting is personal when it is shared
- change several settings before validating any of them
- forget to refresh the affected page after saving
- edit a runtime variable but test the wrong plugin or project
- configure a queue name that does not exist in the scheduler
- change a submit template without checking the generated job script behavior
- modify settings in a test deployment and expect production to change

---

## If a settings change does not persist

Check:

- whether the save request succeeded
- whether the page reloaded stale data
- whether your user role has permission to change that section
- whether browser or API errors appeared during save
- whether another source of configuration is overriding the expected value
- whether the backend logs show a validation or permission error

Useful recovery steps:

1. refresh the Settings page
2. confirm that the value is still present
3. validate with the smallest affected workflow
4. inspect browser network requests if the value disappears
5. check backend logs if the API reports an error
