---
name: Bundle AsyncAPI documents and generate output
description: Merge multi-file AsyncAPI documents into one, then run a generator template to produce docs or code.
api: openapi/asyncapi-server-api-openapi.yml
operations: [bundle, generate, validate]
generated: '2026-09-06'
method: generated
source: openapi/asyncapi-server-api-openapi.yml
---

# Bundle AsyncAPI documents and generate output

Multi-file specs are normal in event-driven work — shared message schemas, per-domain channel files. Bundle first, generate second.

## Steps

1. **Bundle** — `POST /bundle` (operationId `bundle`).

   ```
   POST https://api.asyncapi.com/v1/bundle
   Content-Type: application/json
   {
     "asyncapis": [<document>, <document>],
     "base": <document>
   }
   ```

   - `asyncapis` (required) — the array of documents to merge.
   - `base` — optional document whose `info`, `servers` and top-level metadata the bundle inherits.

   Returns `{"bundled": <document>}` — one self-contained document with references inlined.

2. **Validate the bundle** — `POST /validate` before generating. Generation against an invalid bundle fails late and less legibly.

3. **Generate** — `POST /generate` (operationId `generate`).

   ```
   POST https://api.asyncapi.com/v1/generate
   Content-Type: application/json
   {
     "asyncapi": <bundled document>,
     "template": "@asyncapi/html-template",
     "parameters": { }
   }
   ```

   - `template` (required) — the generator template package. The contract enumerates a set including `@asyncapi/html-template`, `@asyncapi/markdown-template`, `@asyncapi/nodejs-template`, `@asyncapi/java-spring-template`, `@asyncapi/dotnet-nats-template` and `@asyncapi/minimaltemplate`.
   - `parameters` — template-specific parameters. These are NOT validated by the API (the spec says so in a comment), so a wrong parameter name is silently ignored rather than rejected.
   - `use-fallback-generator` — boolean, default false; pins generator 1.17.25 for templates that have not migrated.

   The response is a binary/zip payload of the generated output.

4. **On `422`**, re-read `template`: an unknown template package is the most common cause.

## Notes

- Generation is the heaviest operation on a free, unmetered, shared service. Bundle and validate locally with the CLI when you are iterating, and call the hosted API for the final run.
- The equivalent local flow is `asyncapi bundle` then `asyncapi generate fromTemplate <file> <template>`.

## Related

- `cli/asyncapi-cli.yml` — the local equivalents of every step here
- `packages/asyncapi-packages.yml` — the template packages and their release dates
