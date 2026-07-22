---
name: prisma-orm
description: Guides Prisma ORM usage — schema definition, migrations, type-safe queries, avoiding N+1, transactions, connection pooling for Lambda, and seed patterns. Use whenever code interacts with the database through Prisma. Do not use for raw SQL or other ORMs.
---

# Prisma ORM

## Procedures

**When defining or modifying the Prisma schema**

1. Define models in `prisma/schema.prisma`; use PascalCase for model names and camelCase for field names.
2. Always define an `id` field (prefer `@id @default(cuid())` for new models or `@id @default(autoincrement())` for integer PKs).
3. Add `createdAt DateTime @default(now())` and `updatedAt DateTime @updatedAt` to every entity model.
4. Use explicit relation fields with `@relation` when the inferred relation is ambiguous or when there are multiple relations between the same models.
5. Add database indexes for fields used in `where` clauses that are not already covered by a unique constraint: `@@index([fieldName])`.
6. After modifying `schema.prisma`, run `pnpm run db:migrate:dev` (dev) or `pnpm run db:migrate` (production) to apply changes.
7. Run `pnpm run db:generate` after schema changes to regenerate the Prisma Client types.

**When writing queries**

1. Use the generated Prisma Client (`PrismaClient`) — never write raw SQL unless `prisma.$queryRaw` is explicitly required for a complex query.
2. Avoid N+1 queries: use `include` or `select` to load related data in a single query rather than iterating and querying inside a loop.
3. Use `select` to fetch only the fields needed by the caller; avoid loading full records when only a subset of fields is used.
4. Use `findUniqueOrThrow` and `findFirstOrThrow` when the record must exist — they throw `PrismaClientKnownRequestError` (P2025) automatically instead of returning `null`.
5. Use `upsert` for idempotent create-or-update operations; avoid a separate `findUnique` + conditional `create`/`update` pattern.
6. Wrap multiple dependent mutations in `prisma.$transaction([...])` (sequential) or `prisma.$transaction(async (tx) => { ... })` (interactive) to ensure atomicity.
7. In interactive transactions, use the `tx` client parameter for all queries within the transaction block.

**When deploying in Lambda (connection pooling)**

1. Do not instantiate `PrismaClient` at the module level in Lambda handlers — use a singleton pattern with lazy initialization to reuse connections across warm invocations.
2. Configure the database URL with connection pooling via PgBouncer or Prisma Accelerate when the Lambda function has high concurrency; raw connections to PostgreSQL are limited.
3. Always call `prisma.$disconnect()` in a `finally` block or lifecycle hook when running outside Lambda (scripts, tests) to close the connection pool.

**When seeding**

1. Place seed data in `prisma/seed.ts`; run via `prisma db seed` or the project's `db:seed` script.
2. Use `upsert` in seed files so they are idempotent and can be re-run safely.

## Error Handling

1. Catch `PrismaClientKnownRequestError` and check `error.code` to handle specific database errors:
   - `P2002` — unique constraint violation (duplicate record)
   - `P2025` — record not found
   - `P2003` — foreign key constraint failure
2. When a transaction fails, Prisma rolls back automatically; re-throw the error or map it to a domain error for the caller.
3. When schema drift errors appear (`P3005`, `P3006`), run `pnpm run db:migrate:status` to inspect pending migrations before attempting fixes.
