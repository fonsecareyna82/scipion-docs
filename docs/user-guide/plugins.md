---
hide:
  - toc
---

# Plugins

Plugins extend Scipion capabilities and may expose actions both in the backend and in the web interface.

## Typical plugin-related tasks

In normal use, you may need to:

- review available plugins
- check whether a plugin is already installed
- start an installation or removal flow
- monitor plugin-related task progress
- confirm that the UI reflects the new plugin state

## What to verify before changing plugin state

- you are working in the intended environment
- the backend and worker are running correctly
- Redis and Celery are healthy
- the plugin action is allowed for your role or workflow

## During plugin operations

Pay attention to:

- task status changes
- installation or removal logs
- whether the plugin becomes available only after backend reload or restart

## If a plugin operation gets stuck

Check:

- worker logs
- Redis availability
- task status endpoint responses
- whether the plugin introduced environment-specific dependency issues
