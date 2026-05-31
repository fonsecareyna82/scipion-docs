# Environment Variables in Settings

ScipionWeb includes an environment variables section inside **Settings**. This area lets authorized users review, add, and modify environment variables used by the web application and its runtime integration with Scipion plugins.

Use this page when an external program is already installed on the server and you only need ScipionWeb to point a plugin to that existing installation.

<img src="../../assets/images/screenshots/user-guide/environment.png" alt="Environment variables settings" style="display:block;width:100%;max-width:980px;height:auto;margin:1.2rem auto 1.6rem;" />

*Environment variables managed from ScipionWeb Settings.*

---

## What this page covers

This page describes environment variables managed from the ScipionWeb interface.

It is different from the server-side `.env` file used to configure ScipionAPI itself. For the low-level runtime file, see [Environment Variables (`.env`)](../configuration/env.md).

The Settings environment section is intended for variables that ScipionWeb and plugin execution need to read at runtime, especially path variables used by external scientific software.

---

## Why this is useful

Some Scipion plugins can use external software that has already been installed outside the plugin installation process.

In those cases, you may not want the plugin installer to download or install binaries again. Instead, you can:

1. install only the Python package for the plugin
2. define or update the environment variable expected by that plugin
3. point the variable to the existing external software installation
4. let ScipionWeb pick up the value without requiring a full service restart

This is useful for deployments where large external packages are already managed by administrators, modules, shared storage, containers, or site-specific installation policies.

---

## Hot runtime assimilation

ScipionWeb is able to assimilate these variables while the system is running. After a variable is created or modified from Settings, the updated value becomes available to the relevant ScipionWeb runtime context without forcing users to manually edit server files.

!!! note "Validate after changing variables"
    Even when ScipionWeb picks up the value dynamically, you should still validate the affected plugin or protocol after changing an environment variable.

---

## Common plugin variable pattern

Many plugins expect a path variable with a name that follows this pattern:

```text
PLUGIN_HOME
```

For a real plugin, the prefix normally matches the plugin or external program name. The variable value should point to the directory where the external software is installed.

Examples:

```text
SOMEPLUGIN_HOME=/opt/software/someplugin
EXTERNALTOOL_HOME=/shared/apps/externaltool
MYPROGRAM_HOME=/home/scipion/software/myprogram
```

The exact variable name depends on the plugin. In many cases, the plugin installation or plugin documentation already defines the expected variable name.

---

## Recommended workflow for external software

Use this flow when the external software already exists on the server:

1. Confirm that the external software is installed and accessible from the ScipionWeb host.
2. Install only the Python package for the Scipion plugin, avoiding binary installation if that is the intended deployment strategy.
3. Open **Settings** in ScipionWeb.
4. Go to the environment variables section.
5. Add or update the plugin path variable, usually following the `PLUGIN_HOME` pattern.
6. Save the variable.
7. Run a small validation workflow or open a protocol that depends on the plugin.
8. Check logs if the plugin still cannot locate the external binary.

---

## Add a new environment variable

1. Open **Settings**.
2. Locate the environment variables section.
3. Select the action to add a new variable.
4. Enter the variable name exactly as expected by the plugin.
5. Enter the full filesystem path to the external software installation.
6. Save the variable.
7. Verify that the variable appears in the list.

Before saving, check:

- the variable name uses the correct uppercase spelling
- the path exists on the server where ScipionWeb runs
- the user running ScipionWeb has permission to read and execute files from that path
- the path points to the software installation root expected by the plugin

---

## Modify an existing variable

Use modification when the external software was moved, upgraded, or replaced.

1. Open **Settings**.
2. Find the existing environment variable.
3. Change the value to the new path.
4. Save the change.
5. Validate the affected plugin or protocol.

!!! warning "Changing paths can affect running work"
    Avoid changing a variable used by active workflows unless you understand the impact. A running or queued protocol may depend on the previous value.

---

## Good practice

- keep variable names uppercase and consistent with plugin expectations
- use absolute paths, not relative paths
- avoid pointing variables to user-specific temporary folders
- document externally managed software versions outside ScipionWeb when needed
- test a minimal protocol after changing a plugin path
- avoid duplicating binary installations when a stable external installation already exists

---

## Common mistakes

Users often run into problems when they:

- use the wrong variable name
- point the variable to the binary file instead of the expected installation directory
- point to a path that exists only on their local workstation, not on the server
- forget that the ScipionWeb service user needs filesystem permissions
- install plugin binaries unnecessarily even though the external software is already available
- update a variable while another workflow is still using the previous path

---

## If a plugin still cannot find the software

Check:

- the exact variable name expected by the plugin
- the saved value in ScipionWeb Settings
- whether the path exists on the ScipionWeb server
- whether the service user has read and execute permissions
- whether the plugin expects the root folder, a `bin` folder, or another specific subdirectory
- whether the protocol logs show the environment value being used

Useful recovery steps:

1. verify the external software path manually on the server
2. correct the variable in Settings
3. run a small plugin-specific validation protocol
4. inspect protocol and backend logs if the path is still not detected
