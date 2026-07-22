---
name: context7
description: >-
  Retrieves authoritative, up-to-date technical documentation, API references,
  configuration details, and code examples for any developer technology.

  Use this skill whenever answering technical questions or writing code that
  interacts with external technologies. This includes libraries, frameworks,
  programming languages, SDKs, APIs, CLI tools, cloud services, infrastructure
  tools, and developer platforms.

  Common scenarios:
  - looking up API endpoints, classes, functions, or method parameters
  - checking configuration options or CLI commands
  - answering "how do I" technical questions
  - generating code that uses a specific library or service
  - debugging issues related to frameworks, SDKs, or APIs
  - retrieving setup instructions, examples, or migration guides
  - verifying version-specific behavior or breaking changes

  Prefer this skill whenever documentation accuracy matters or when model
  knowledge may be outdated.
---

# Documentation Lookup

Retrieve current documentation and code examples for any library using the Context7 CLI.

Make sure the CLI is up to date before running commands:

```bash
npm install -g ctx7@latest
```

Or run directly without installing:

```bash
npx ctx7@latest <command>
```

## Workflow

Two-step process: resolve the library name to an ID, then query docs with that ID.

```bash
# Step 1: Resolve library ID
ctx7 library <name> <query>

# Step 2: Query documentation
ctx7 docs <libraryId> <query>
```

You MUST call `ctx7 library` first to obtain a valid library ID UNLESS the user explicitly provides a library ID in the format `/org/project` or `/org/project/version`.

IMPORTANT: Do not run these commands more than 3 times per question. If you cannot find what you need after 3 attempts, use the best result you have.

## Step 1: Resolve a Library

Resolves a package/product name to a Context7-compatible library ID and returns matching libraries.

```bash
ctx7 library fastify "How to define route schema validation"
ctx7 library prisma "How to define one-to-many relations with cascade delete"
ctx7 library serverless-framework "How to configure Lambda function with SQS trigger"
```

Always pass a `query` argument — it is required and directly affects result ranking. Use the user's intent to form the query, which helps disambiguate when multiple libraries share a similar name. Do not include any sensitive or confidential information such as API keys, passwords, credentials, personal data, or proprietary code in your query.

### Result fields

Each result includes:

- **Library ID** — Context7-compatible identifier (format: `/org/project`)
- **Name** — Library or package name
- **Description** — Short summary
- **Code Snippets** — Number of available code examples
- **Source Reputation** — Authority indicator (High, Medium, Low, or Unknown)
- **Benchmark Score** — Quality indicator (100 is the highest score)
- **Versions** — List of versions if available.

### Selection process

1. Analyze the query to understand what library/package the user is looking for
2. Select the most relevant match based on name similarity, description relevance, documentation coverage, source reputation, and benchmark score
3. If multiple good matches exist, acknowledge this but proceed with the most relevant one
4. If no good matches exist, clearly state this and suggest query refinements

## Step 2: Query Documentation

Retrieves up-to-date documentation and code examples for the resolved library.

```bash
ctx7 docs /fastify/fastify "How to add JSON schema validation to a route"
ctx7 docs /prisma/prisma "How to define one-to-many relations with cascade delete"
ctx7 docs /serverless/serverless "How to configure SQS event source mapping"
```

### Writing good queries

The query directly affects the quality of results. Be specific and include relevant details.

| Quality | Example |
| ------- | ------- |
| Good    | `"How to set up authentication with JWT in Fastify"` |
| Good    | `"Prisma transaction with rollback on error"` |
| Bad     | `"auth"` |
| Bad     | `"database"` |

## Authentication

Works without authentication. For higher rate limits:

```bash
# Option A: environment variable
export CONTEXT7_API_KEY=your_key

# Option B: OAuth login
ctx7 login
```

## Error Handling

If a command fails with a quota error ("Monthly quota reached" or "quota exceeded"):

1. Inform the user their Context7 quota is exhausted
2. Suggest they authenticate for higher limits: `ctx7 login`
3. If they cannot or choose not to authenticate, answer from training knowledge and clearly note it may be outdated

Do not silently fall back to training data — always tell the user why Context7 was not used.

## Common Mistakes

- Library IDs require a `/` prefix — `/fastify/fastify` not `fastify/fastify`
- Always run `ctx7 library` first — `ctx7 docs fastify "hooks"` will fail without a valid ID
- Use descriptive queries, not single words — `"Fastify route lifecycle hooks"` not `"hooks"`
- Do not include sensitive information (API keys, passwords, credentials) in queries
