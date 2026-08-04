---
name: serverless-aws-lambda
description: Guides AWS Lambda development with TypeScript across both IaC stacks in use at Bliss — Serverless Framework (serverless.ts) and AWS CDK (@saude-bliss/bliss-aws-cdk-modules catalog). Covers handler signatures, event sources (API Gateway, SQS, SNS, S3, EventBridge), environment variables, secrets, IAM, stages, esbuild packaging, cold start mitigations. Do not use for non-Lambda deployments.
---

# AWS Lambda (Serverless Framework or AWS CDK)

Bliss runs Lambda behind **two IaC stacks, both valid**. Everything under "Lambda functions" and "Error Handling" applies to both; the configuration procedures are stack-specific. Never mix stacks in one service, and never migrate a service from one to the other as part of a feature.

## Procedures

**When starting work on a Lambda repository — identify the stack first**

1. `serverless.ts` (or `serverless.yml`) at the repo root → **Serverless Framework**. Stages are `sandbox` / `production`. API Gateway is REST API **v1** by default. Reference: `bliss-insurer-edge`, `bliss-order-gateway`.
2. A `bliss-cdk/` folder containing `catalog/` → **AWS CDK**. Environments are `dev` / `prod` (via `-c environment=`). API Gateway is HTTP API **v2**. Reference: `bliss-auth-gateway`.
3. Never assume from the repo age or from a sibling service. Check the files.
4. For a **new** service, prefer CDK; choose Serverless Framework only for a concrete reason (`serverless-offline` in the local flow, mandatory parity with a sibling Serverless service, or an event source the CDK catalog does not support — see the SQS note below). Record the choice and its justification in the TechSpec.

**When creating or modifying Lambda functions (both stacks)**

1. Use the handler signature matching the event source, and mind that the API Gateway payload version differs between stacks:
   - API Gateway, Serverless Framework (`http`, REST v1): `APIGatewayProxyHandler` → `APIGatewayProxyResult`
   - API Gateway, CDK (`trigger: http`, HTTP API v2): `APIGatewayProxyEventV2` → `APIGatewayProxyResultV2`
   - SQS: `SQSHandler` → `void` (throw to trigger DLQ)
   - SNS: `SNSHandler` → `void`
   - S3: `S3Handler` → `void`
   - EventBridge: `EventBridgeHandler<DetailType, Detail>` → `void`
2. Import types from the `aws-lambda` package: `import type { APIGatewayProxyHandler } from 'aws-lambda'`.
3. Export the handler under the name the IaC declares: Serverless Framework points at `handler.main` (`export const main = ...`); the CDK catalog points at `entryPoint` inside `handlerFile` (which defaults to `src/handlers/{name}.ts`).
4. Keep handlers thin — parse event, call service, return response. Business logic belongs in service modules under the same function directory.
5. Parse and validate event body before processing; never trust raw Lambda event input.
6. Use environment variables via `process.env` for all configuration; never hardcode credentials, ARNs, or endpoint URLs.
7. Set memory and timeout per function; never leave every function on the same default.
8. Cache expensive setup (framework app, DB client, loaded secrets) in module scope so it is paid once per execution environment. When the app hydrates env vars from a secret at cold start, resolve the secret **before** importing any module that reads required env vars — use a dynamic `import()` for that module, never a static top-level one, or the cold start crashes on a missing variable.
9. For SQS consumers, configure `batchSize` and `functionResponseType: 'ReportBatchItemFailures'` to enable partial batch failure handling.
10. Use Lambda Layers for large shared dependencies (Prisma client, Playwright) to avoid bundle size limits and speed up deploys.

**When handling errors in Lambda (both stacks)**

1. For API Gateway handlers, always return a structured JSON error response — never let unhandled exceptions produce an empty or malformed response.
2. For SQS/SNS/S3 handlers, throw on unrecoverable errors to route messages to the DLQ; catch and log transient errors with retry logic.
3. Log structured JSON to stdout (CloudWatch picks it up automatically); include `requestId`, `functionName`, and relevant context in every log entry.

**When configuring Serverless Framework**

1. Keep `serverless.ts` typed using `import type { AWS } from '@serverless/typescript'`.
2. Organize functions by domain — each function in its own file, exported and imported into the main config.
3. Declare all required environment variables under `provider.environment` or per-function `environment`; document them in `.env.example`.
4. Declare IAM permissions under `provider.iam.role.statements` with least privilege: exact `Resource` ARNs, minimum `Action` set.
5. Use stages (`sandbox`, `production`) to separate environments; access via `${sls:stage}`.
6. Use `${ssm:/path/to/param}` or Secrets Manager references for sensitive values; never commit plaintext secrets.
7. Package functions individually with the `serverless-esbuild` plugin to minimize cold start size; mark AWS SDK packages as external since they ship in the Lambda runtime.
8. Test locally with `serverless offline`, or invoke a single handler with `serverless invoke local`.
9. Deploy with `pnpm run deploy:staging` / `pnpm run deploy:production`.

**When configuring the AWS CDK catalog (`@saude-bliss/bliss-aws-cdk-modules`)**

1. One `catalog/functions/*.yaml` per Lambda. The schema is `.strict()` — an unknown or misspelled field fails the synth with the file and field named. When editing from an example, replace the whole file; do not paste snippets into an existing one (duplicate `defaults:` keys are a common `YAMLParseError`).
2. Required fields: `name`, `entryPoint` (the **export name**), and `trigger`.
3. Available triggers: `http`, `topic` (SNS — `topicTriggerName` creates, `topicArn` imports), `schedule` (EventBridge cron, timezone defaults to `America/Sao_Paulo`), `bucket` (S3 object created, via `triggerBucket`). **There is no SQS trigger** — an SQS consumer needs the `native` escape hatch or the Serverless Framework stack. Settle this before choosing CDK for a queue-driven service.
4. Tunables go under `defaults` and are selectively overridden per environment under `environments.<env>`: `memoryMb`, `timeoutSeconds`, `reservedConcurrency`, `environmentVariables`, `secretsEnvironmentVariables`, `tags`, `vpc`, `customDomain`. Merge semantics: **scalars override, maps merge key by key, lists concatenate and dedupe.**
5. `${VAR}` in the YAML interpolates an environment variable at synth time. Every `${VAR}` must be exported wherever `cdk synth`/`deploy` runs — in CI, as a repo secret or variable wired into the workflow `env:`. A missing one fails with the variable named.
6. `vpc` and `customDomain` take **explicit IDs** (`vpcId`/`subnetIds`/`securityGroupIds`, `domainName`/`hostedZoneId`/`certificateArn`) — never `fromLookup`, which is what keeps `cdk synth` credential-free. The VPC, hosted zone, and ACM certificate must already exist; the library does not create them.
7. Choose the HTTP shape deliberately:
   - **One Lambda per endpoint** (preferred for a REST surface): declare `httpRoute: {method, path}` on each function. All functions with `httpRoute` share a single HTTP API, each with its own route visible in the console. `path` uses API Gateway syntax (`{id}`), **not** Fastify/Express `:id`.
   - **Whole app in one Lambda**: omit `httpRoute`; `trigger: http` then creates a dedicated API with a `$default` catch-all and the framework routes internally. Convenient for adopting an existing app unchanged.
   - With `httpRoute`, the custom domain belongs to the shared API in `catalog/http-api.yaml` (a single file at the catalog root, not a folder). Declaring `customDomain` on a function that also has `httpRoute` is a validation error.
   - Both shapes coexist in one catalog without conflict.
8. Declare IAM with `iamStatements: [{actions, resources}]`, least privilege, exact ARNs.
9. Secrets — the library creates an **empty** container; the value is never versioned:
   - `secrets: [name]` → creates the secret and grants read to the function role. Creates **no** env var.
   - `secretsEnvironmentVariables: [name]` → grants read **and** creates env var `NAME_IN_UPPER_SNAKE` whose **value is the secret name**, not its content. The code resolves the real value at runtime via the SDK — Lambda does not inject secret values at deploy time (unlike ECS).
   - When the app already reads a fixed env var name (e.g. `AWS_SECRET_NAME` pointing at one JSON secret with many keys), combine `secrets: [name]` with a manual `environmentVariables: { AWS_SECRET_NAME: name }` — `secretsEnvironmentVariables` derives the env var name automatically and cannot be renamed.
   - Populate the value once after the first deploy: `aws secretsmanager put-secret-value --secret-id <name> --secret-string '<json>'`.
   - Create a shared secret in exactly one function's `secrets:`, and have the others import it by name via `secretsEnvironmentVariables:` — this preserves its CloudFormation logical ID so an already-populated secret is not recreated empty.
10. Keep the CDK isolated in its own folder (`bliss-cdk/`) with its own `package.json` and lockfile when the repo is not a workspace. If the repo **is** a pnpm workspace, a subfolder is *not* isolated — pnpm walks up to the nearest `pnpm-workspace.yaml` and ignores the subfolder's `packageManager`/`onlyBuiltDependencies`. Prefer a separate repository in that case.
11. Set `NODE_ENV: production` in `environmentVariables` for any service whose logger picks a dev transport off `NODE_ENV` — Lambda never sets it, and a `pino-pretty` transport does not survive the single-file esbuild bundle.
12. Bootstrap the account/region once with `pnpm exec cdk bootstrap`. `synth` and `diff` need no AWS credentials; `deploy` and `destroy` do.
13. Deploy is GitOps, matching `bliss-infra-modules`: PR → `cdk diff` (dev); push on `main` → `cdk deploy` (dev); tag `v*` → `cdk deploy` (prod). `@saude-bliss/bliss-aws-cdk-modules` is a restricted package — installs need `NPM_READ_TOKEN` in CI and an authenticated `~/.npmrc` locally.
14. Reach for `native` (raw CDK props, merged last over the construct defaults) only when the neutral YAML contract cannot express the field, and say why in a comment.

## Error Handling

1. When cold starts are unacceptably slow, audit bundle size (`serverless package` + inspect `.serverless/*.zip`, or the CDK asset), externalize heavy dependencies to a Layer, and verify `esbuild` tree-shaking is active.
2. When SQS messages are not being processed, verify the IAM role has `sqs:ReceiveMessage`, `sqs:DeleteMessage`, and `sqs:GetQueueAttributes` on the queue ARN.
3. When environment variables are undefined at runtime, check the declaration scope — `provider` vs function level in `serverless.ts`; `defaults` vs `environments.<env>` in the CDK catalog — and confirm the stage/environment is the one you think it is. The names differ between stacks (`sandbox`/`production` vs `dev`/`prod`), so a URL's name is not proof of which environment it points at.
4. When a secret reads as empty in a CDK function, remember the env var holds the secret **name**: resolve the value via the SDK, and confirm `put-secret-value` has actually been run for that environment.
5. When `cdk synth` fails with `Catálogo inválido em …`, the YAML has an unknown or malformed field — the message names both. When it names an undefined `${VAR}`, export it where the command runs.
6. When the CDK bundle fails with `npx canceled due to missing packages: ["esbuild@…"]`, the handler lives outside `bliss-cdk/`: esbuild runs with `cwd` at the repo root, so the root also needs its own dependencies installed and `esbuild` as an explicit devDependency.
7. When `pnpm install` reports `ERR_PNPM_IGNORED_BUILDS` (and esbuild turns up missing later), the fix depends on the pnpm major — check `pnpm -v` first:
   - **pnpm 10**: `packageManager: pnpm@10.15.0` plus `pnpm.onlyBuiltDependencies: ["esbuild"]` in `package.json`, which is what the library's `init` writes.
   - **pnpm 11**: the `pnpm` field in `package.json` is **no longer read** — it warns and ignores. The setting moved to `pnpm-workspace.yaml` (`allowBuilds: { esbuild: true }`). Mind the side effect: creating that file makes the repo a workspace root, which is exactly what breaks `bliss-cdk/` isolation (see the CDK procedure). When the app itself does not need esbuild — a `tsc`-only build, for instance — prefer leaving esbuild out of the app root entirely and adding it only where the CDK bundling requires it.
   - Either version: `pnpm approve-builds` resolves it interactively. Inside a pnpm workspace, subfolder-level settings are ignored.
8. When an OIDC deploy fails with `Not authorized to perform sts:AssumeRoleWithWebIdentity`, the role's trust policy does not cover the repo's `sub` (GitHub includes immutable org and repo IDs in it). Check with whoever administers the role.
9. When a function in a VPC times out reaching RDS, it is network, not IaC: security group egress/ingress and subnet routes. A function in a VPC also has **no internet** without a NAT Gateway — and creating or destroying it is legitimately slower, because CloudFormation waits on the ENI.
10. Deploy — in either stack — never applies a migration or a seed. Verify the database directly before claiming a feature is live in an environment.
