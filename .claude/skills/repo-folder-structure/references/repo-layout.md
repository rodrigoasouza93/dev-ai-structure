# Repository layout reference — Bliss projects

Two main patterns depending on the project type.

---

## Lambda / Serverless (majority of Bliss projects)

```text
src/
  functions/
    <function-name>/
      handler.ts        ← Lambda entry point (thin: parse event, call service, return)
      index.ts          ← Serverless Framework function config
      schema.ts         ← JSON Schema for request/response validation
      <name>.service.ts ← business logic
      <name>.test.ts    ← unit / integration tests
    <other-function>/
      ...
  libs/
    <lib-name>/         ← shared libraries (e.g. vertex-document-extraction)
      index.ts
      ...
  types/                ← shared TypeScript types
  schemas/              ← shared JSON Schema / Zod schemas
  index.ts or server.ts ← app bootstrap (when applicable)
```

### Serverless function config example (`index.ts`)

```typescript
import type { AWS } from '@serverless/typescript'

const config: AWS['functions'][string] = {
  handler: 'src/functions/myFunction/handler.main',
  events: [
    {
      http: {
        method: 'post',
        path: 'resource',
        cors: true,
      },
    },
  ],
}

export default config
```

### Lambda handler pattern

```typescript
import type { APIGatewayProxyHandler } from 'aws-lambda'
import { MyService } from './my-function.service'

export const main: APIGatewayProxyHandler = async (event) => {
  const body = JSON.parse(event.body ?? '{}')
  const result = await MyService.process(body)
  return {
    statusCode: 200,
    body: JSON.stringify(result),
  }
}
```

### Avoid

- Business logic inside handler files (keep handlers < 30 lines)
- Shared state between Lambda invocations
- Hardcoded credentials or secrets (use environment variables)

---

## Frontend (intranet — React + Vite + Ant Design)

```text
src/
  app/
  assets/
  components/
    ui/                 ← generic design-system primitives (wrapping Ant Design)
    ...
  features/
    <feature-name>/
      components/
      hooks/
      lib/
      api/
      types.ts
      index.ts
  hooks/                ← cross-feature hooks
  lib/                  ← generic helpers
  pages/                ← routed page components
  routes/               ← route definitions
  types/                ← global TypeScript types
  main.tsx
  index.css
```

### Avoid

- Scattered HTTP calls in components without a clear `features/<x>/api` layer
- Heavy domain logic inside `components/ui`; primitives stay dumb and reusable
- Importing Ant Design components directly in feature files without a wrapper when customization is needed

---

## Backend (non-Lambda, e.g. broker-gateway)

Three-layer layout:

| Folder | Responsibility |
|--------|----------------|
| `controllers/` | HTTP handlers: route wiring, parsing, status codes — delegate to services |
| `services/` | Domain rules, orchestration, validations |
| `data/` | External IO: repositories, outbound HTTP/SDKs, queues, persistence |

```text
src/
  controllers/
    <resource>.controller.ts
  services/
    <domain>.service.ts
  data/
    repositories/
    clients/
  types/
  schemas/
  index.ts or server.ts
```
