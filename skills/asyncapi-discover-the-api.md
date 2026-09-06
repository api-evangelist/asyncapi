---
name: Discover what the AsyncAPI Server API exposes
description: Ask the service itself which commands it serves and what version it is running, before hard-coding anything.
api: openapi/asyncapi-server-api-openapi.yml
operations: [help, getVersion]
generated: '2026-09-06'
method: generated
source: openapi/asyncapi-server-api-openapi.yml
---

# Discover what the AsyncAPI Server API exposes

The service publishes its own capability index. Use it instead of assuming, especially when calling a self-hosted instance that may be older than the hosted one.

## Steps

1. **List the commands** — `GET /help` (operationId `help`).

   ```
   GET https://api.asyncapi.com/v1/help
   ```

   Returns an array of `{command, url}` — observed 2026-09-06: `version`, `validate`, `parse`, `generate`, `convert`, `bundle`, `help`. Note that `diff` is documented in the OpenAPI but did not appear in the live help list; treat `/help` as the authoritative answer for the instance you are actually calling.

2. **Describe one command** — `GET /help/{command}` (same operationId `help`, with the command in the path).

   ```
   GET https://api.asyncapi.com/v1/help/validate
   ```

   Returns `{command, method, summary, requestBody}` — enough to build the call without the OpenAPI in hand.

3. **Check what is running** — `GET /version` (operationId `getVersion`). Returns the AsyncAPI CLI version, name, description, a `runtime` block (node version, environment, platform, arch, uptime, start time), `repository` URLs and licence, and an `api` block whose `health` is one of `ok`, `degraded`, `error`.

4. **Use `api.health` as the liveness signal.** There is no status page for this service — `status.asyncapi.com` does not resolve — so `GET /version` is the only health signal the provider offers.

## Notes

- A `404` from `/help/{command}` means the command does not exist on that instance, and comes back as the standard `Problem` envelope.
- Self-hosted instances run whatever `@asyncapi/server-api` or `@asyncapi/cli` version you installed; `GET /version` is how you tell them apart from the hosted deployment.

## Related

- `lifecycle/asyncapi-lifecycle.yml` — versioning and the archived server-api repo
- `authentication/asyncapi-authentication.yml` — why none of these calls need a credential
