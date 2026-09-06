---
name: Validate an AsyncAPI document
description: Check an AsyncAPI document against the specification and read the diagnostics, using the free hosted AsyncAPI Server API.
api: openapi/asyncapi-server-api-openapi.yml
operations: [validate, parse]
generated: '2026-09-06'
method: generated
source: openapi/asyncapi-server-api-openapi.yml
---

# Validate an AsyncAPI document

Use this before publishing, in CI, or whenever you are handed a document you did not write.

## Before you call

- Base URL is `https://api.asyncapi.com/v1`. No API key, no signup, no Authorization header — an anonymous call is the intended call.
- No rate limits are published and no `RateLimit-*` headers come back, so back off on your own schedule and do not assume a budget.
- Every operation is side-effect free. Nothing you send is stored, so a retry is safe — but there is no `Idempotency-Key` contract, so do not claim one.

## Steps

1. **Validate** — `POST /validate` (operationId `validate`).
   Body: `{"asyncapi": <document>}` where `<document>` is the AsyncAPI document as an object, a YAML/JSON string, or a URL the server can fetch.

   ```
   POST https://api.asyncapi.com/v1/validate
   Content-Type: application/json
   {"asyncapi": {"asyncapi": "3.1.0", "info": {"title": "Orders", "version": "1.0.0"}, "channels": {}}}
   ```

2. **Read the response, not just the status.** The published contract declares `204 No Content` for a valid document, but the deployed service returns `200` with a body carrying `status` (`valid` / `invalid`), `diagnostics[]` and a numeric `score` (0–100). Code for the body; treat 204 as also-valid.

3. **On `400` or `422`**, the body is the shared `Problem` envelope — `type`, `title`, `status`, plus optional `detail` and `instance`. It is RFC 7807-shaped but served as `application/json`, not `application/problem+json`, so do not content-negotiate for it. `400` means the document is invalid; `422` means a request parameter was rejected.

4. **Need the resolved document rather than a verdict?** Call `POST /parse` (operationId `parse`) with the same `{"asyncapi": ...}` body. It returns `{"parsed": ...}` — the document with `$ref`s resolved — which is what you want before analysing channels or messages programmatically.

## Notes

- Diagnostics include advisory rules, not only errors. A document can be `valid` and still report `asyncapi-latest-version` telling you 3.1.0 is available.
- The same check runs locally as `asyncapi validate <file>` if you would rather not send documents to a third-party host.

## Related

- `errors/asyncapi-problem-types.yml` — the error envelope
- `conventions/asyncapi-conventions.yml` — including the 204-vs-200 drift recorded above
