---
name: fastify-rest-http
description: Enforces Fastify route definitions, JSON Schema validation, proper HTTP status codes, plugin architecture, error handling with setErrorHandler, and native fetch for external HTTP calls. Do not use when the HTTP framework is not Fastify.
---

# Fastify REST / HTTP

## Procedures

**When defining routes and HTTP handlers in Fastify**

1. Register routes using `fastify.get`, `fastify.post`, `fastify.put`, `fastify.patch`, `fastify.delete` with explicit method naming.
2. Define request and response shapes using JSON Schema on the route `schema` property; prefer `ajv` validation through Fastify's built-in serialization.
3. Use plural, lowercase, kebab-case resource names: `/health-plans`, `/broker-managers`, `/quotation-jobs`.
4. Prefer POST for mutations that do not map to idempotent verbs; use action-noun paths for non-CRUD operations: `POST /orders/:id/cancel`.
5. Return correct HTTP status codes consistently:
   - `200` — successful read or update
   - `201` — resource created
   - `204` — success with no body
   - `400` — malformed request / validation error
   - `401` — unauthenticated
   - `403` — unauthorized
   - `404` — resource not found
   - `422` — business rule violation
   - `500` — internal server error
6. Use `reply.code(status).send(body)` to set status and body explicitly; never rely on implicit status codes.
7. Encapsulate related routes inside Fastify plugins using `fastify.register(plugin, { prefix: '/resource' })`.
8. Register a centralized error handler via `fastify.setErrorHandler` to normalize error responses; do not handle errors individually in each route unless the behavior is route-specific.
9. Use native `fetch` (Node 18+) for outbound HTTP calls to external services; avoid `axios` or `node-fetch` unless the project already depends on them.
10. Keep route handlers thin: validate input, call a service, return the result. Business logic belongs in service modules.
11. Use `fastify.addHook` for cross-cutting concerns (auth, logging, tracing) rather than repeating logic in each route.
12. Document routes using the JSON Schema `description` and `tags` fields for OpenAPI generation when the project uses `@fastify/swagger`.

## Error Handling

1. When Fastify's schema validation rejects a request, it returns a `400` by default; customize the error message via `setErrorHandler` or `schemaErrorFormatter` if the format must match the API contract.
2. When a plugin fails to register, check that `async/await` is used correctly and that all dependencies are awaited before the plugin resolves.
3. When route-level type inference fails in TypeScript, supply explicit generics: `fastify.get<{ Params: ParamsType; Body: BodyType }>('/path', handler)`.
