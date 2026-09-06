---
name: Detect breaking changes between two AsyncAPI documents
description: Diff a proposed AsyncAPI document against the deployed one before publishing, and read the result.
api: openapi/asyncapi-server-api-openapi.yml
operations: [diff, validate]
generated: '2026-09-06'
method: generated
source: openapi/asyncapi-server-api-openapi.yml
---

# Detect breaking changes between two AsyncAPI documents

Run this in the pull request that changes an event contract, before consumers find out at runtime.

## Steps

1. **Validate both documents first** — `POST /validate` (operationId `validate`) on each. A diff against an invalid document tells you nothing useful.

2. **Diff** — `POST /diff` (operationId `diff`).

   ```
   POST https://api.asyncapi.com/v1/diff
   Content-Type: application/json
   {"asyncapis": [<old document>, <new document>]}
   ```

   Order matters: the first element is the baseline, the second is the candidate. The response is `{"diff": ...}`.

3. **Act on the diff.** Removed channels, removed operations, removed message properties and tightened required-field lists are the changes that break consumers. Additions generally do not.

4. **Fail the build on a breaking diff.** The API returns no verdict field — it returns the differences — so the policy about what counts as breaking is yours to encode, not the service's.

## Notes

- Both documents are sent in full on every call; there is no stored baseline to compare against. If you want a persistent baseline, keep the previous document in your own repository and send it each time.
- The local equivalent is `asyncapi diff OLD NEW`.

## Related

- `conventions/asyncapi-conventions.yml` — statelessness, retries and error handling
- `lifecycle/asyncapi-lifecycle.yml` — how AsyncAPI itself announces its own breaking changes
