---
name: Upgrade an AsyncAPI document to the current spec version
description: Convert a 2.x AsyncAPI document (or an OpenAPI 3.0 document) to AsyncAPI 3.1.0 and confirm the result validates.
api: openapi/asyncapi-server-api-openapi.yml
operations: [convert, validate, diff]
generated: '2026-09-06'
method: generated
source: openapi/asyncapi-server-api-openapi.yml
---

# Upgrade an AsyncAPI document to the current spec version

AsyncAPI 3.1.0 shipped 2026-01-31. `POST /convert` moves 2.x documents forward, and also converts OpenAPI 3.0 documents into AsyncAPI.

## Steps

1. **Convert** — `POST /convert` (operationId `convert`).

   ```
   POST https://api.asyncapi.com/v1/convert
   Content-Type: application/json
   {
     "source": <document>,
     "format": "asyncapi",
     "target-version": "3.1.0"
   }
   ```

   - `source` (required) — an AsyncAPI document or an OpenAPI 3.0 document.
   - `format` (required) — `asyncapi` or `openapi`; declares what `source` is.
   - `target-version` — one of `2.x`, `3.0.0`, `3.1.0`, default `latest`. Only meaningful when `format` is `asyncapi`.
   - `perspective` — `server` or `client`, default `server`. Only meaningful when converting from OpenAPI: it decides whether the generated AsyncAPI describes what you publish or what you consume.

   The response is `{"converted": <document>}`.

2. **Validate the result** — `POST /validate` (operationId `validate`) with the converted document. Do not ship a conversion you have not re-validated; conversion is mechanical and can surface constructs that were legal in 2.x and are not in 3.x.

3. **Diff old against new** — `POST /diff` (operationId `diff`) with `{"asyncapis": [<old>, <new>]}` to see exactly what the upgrade moved. This is the step that tells your consumers what changed.

## Notes

- `422` on convert usually means an unsupported `target-version` or a `perspective` on a non-OpenAPI source, not a bad document.
- The 2.x-to-3.x move is a real restructuring — operations were separated from channels in 3.0.0 — so expect the diff to be large even for an unchanged API.

## Related

- `changelog/asyncapi-changelog.yml` — spec version history
- `conformance/asyncapi-conformance.yml` — which spec versions the contract binds to
