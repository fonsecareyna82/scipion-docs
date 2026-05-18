# Deployment Options

ScipionWeb can be deployed in two main ways.

## Integrated mode

Integrated mode is the recommended option for most installations. In this mode, ScipionAPI serves both the API and the compiled Web UI from the same runtime.

This is the simplest option for labs, internal installations, and first-time deployments.

## Separate deployment

In separate deployment mode, the frontend and API are hosted independently.

This option is useful for advanced infrastructure setups, but it requires extra configuration such as CORS, API URL configuration, HTTPS, and reverse proxy rules.

## Recommendation

Use integrated mode unless you have a specific reason to host the frontend separately.

## Related pages

- [Runtime API URL Configuration](runtime-config.md)
- [Integrated Mode](../configuration/integrated-mode.md)
- [Separate Deployment](../configuration/separate-deployment.md)