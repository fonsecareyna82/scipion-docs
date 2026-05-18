# Deployment Options

ScipionWeb can be deployed in two main ways.

## Integrated mode

Integrated mode is the recommended option for most installations. In this mode, ScipionAPI serves both the API and the compiled Web UI from the same runtime.

Use this option for local labs, internal installations, and first-time deployments.

## Separate deployment

In separate deployment mode, the frontend and API are hosted independently.

Use this option only when your infrastructure requires independent frontend and backend hosting.

## Recommendation

Use integrated mode unless you have a specific reason to host the frontend separately.

## Related pages

- [Runtime API URL Configuration](runtime-config.md)
- [Integrated Mode](../configuration/integrated-mode.md)
- [Separate Deployment](../configuration/separate-deployment.md)
