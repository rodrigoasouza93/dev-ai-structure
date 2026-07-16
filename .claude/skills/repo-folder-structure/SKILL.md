---
name: repo-folder-structure
description: Defines folder structure for two layouts — Lambda/Serverless (src/functions/*) and full-stack (features-based frontend + controllers-services-data backend). Applies when creating routes, Lambda handlers, features, or backend modules for consistency. Do not use for monorepo-wide policies outside these templates.
---

# Repository folder structure

## Procedures

**When adding Lambda functions (Serverless Framework projects)**

1. Place each Lambda function under `src/functions/<function-name>/` with its own directory.
2. Inside each function directory, use the standard file layout:
   - `handler.ts` — Lambda handler entry point
   - `index.ts` — Serverless Framework function config
   - `schema.ts` — request/response JSON Schema definitions
   - `<function-name>.service.ts` — business logic (when applicable)
   - `<function-name>.test.ts` — unit/integration tests
3. Shared libraries and utilities live under `src/libs/<lib-name>/`.
4. Shared types and schemas live under `src/types/` or `src/schemas/`.
5. Keep Lambda handlers thin: parse the event, call a service, return the formatted response.
6. Respect the dependency direction: `handler → service → repository/client`.

Read `references/repo-layout.md` for the canonical tree snapshot whenever placement is ambiguous.

**When adding frontend React assets (intranet)**

1. Colocate artifacts that change together: component implementations, narrowly scoped styles, companion tests, and feature-local typings.
2. Place product-domain UI under `src/features/<feature-name>/` with subfolders (`components`, `hooks`, optional `lib`, `api`, `types`), exporting public surfaces via a selective `index.ts` barrel when required.
3. Keep shared primitives and design-system elements under `src/components/ui` and sibling global reusable components without embedding domain rules inside primitive layers.
4. Keep cross-feature hooks under `src/hooks`; keep generic helpers under `src/lib`.
5. Assemble routed experiences under `src/pages`; define route tables under `src/routes`; keep bootstrap concerns under `src/app` alongside `main.tsx` and global styles.
6. Avoid gratuitous nesting beyond roughly three levels; prefer `baseUrl`/path aliases declared in TypeScript configs for readability.

**When extending backend modules (non-Lambda)**

1. Respect the dependency direction `HTTP → controllers/ → services/ → data/`; controllers marshal HTTP shapes and statuses, services encapsulate orchestration and domain guards, data modules perform persistence and integrations.
2. Keep handlers in `controllers/`, workflows in `services/`, adapters in `data/` (`repositories`, `clients`, shared external SDK boundaries).
3. Share cross-layer schemas or validation artifacts under neutral folders (`src/schemas`, `src/types`) avoiding import cycles violating the layering arrow.

## Error Handling

1. When new HTTP calls emerge inside UI primitives, relocate them behind `features/<name>/api` or centralized HTTP clients.
2. When backend logic leaks framing details into controllers, refactor transport mapping upward and domain rules downward per the layered flow.
3. When a Lambda handler grows beyond ~50 lines, extract business logic into a dedicated service file under the same function directory.
