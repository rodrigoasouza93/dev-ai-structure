---
name: serverless-aws-lambda
description: Guides AWS Lambda function development with Serverless Framework and TypeScript — handler signatures, event sources (API Gateway, SQS, SNS, S3, EventBridge), environment variables, IAM, stages, packaging with esbuild, cold start mitigations. Do not use for non-Lambda or non-Serverless-Framework deployments.
---

# Serverless AWS Lambda

## Procedures

**When creating or modifying Lambda functions**

1. Use the standard handler signature for each event source:
   - API Gateway: `APIGatewayProxyHandler` → returns `APIGatewayProxyResult`
   - SQS: `SQSHandler` → returns `void` (throw to trigger DLQ)
   - SNS: `SNSHandler` → returns `void`
   - S3: `S3Handler` → returns `void`
   - EventBridge: `EventBridgeHandler<DetailType, Detail>` → returns `void`
2. Import types from `aws-lambda` package: `import type { APIGatewayProxyHandler } from 'aws-lambda'`.
3. Always export the handler as a named export matching the Serverless Framework `handler` path: `export const main = ...`.
4. Keep handlers thin — parse event, call service, return response. Business logic belongs in service modules under the same function directory.
5. Parse and validate event body before processing; never trust raw Lambda event input.
6. Use environment variables via `process.env` for all configuration; never hardcode credentials, ARNs, or endpoint URLs.
7. Define all required environment variables in `serverless.ts` under `provider.environment` or per-function `environment`; document them in `.env.example`.
8. Declare IAM permissions in `serverless.ts` under `provider.iam.role.statements` with least-privilege: specify exact `Resource` ARNs and minimum required `Action` set.
9. Use stages (`sandbox`, `production`) to separate environments; access via `${sls:stage}` in `serverless.ts`.
10. Package functions individually using `esbuild` bundler (`serverless-esbuild` plugin) to minimize cold start size; mark AWS SDK packages as external since they are available in the Lambda runtime.
11. Set appropriate memory and timeout values per function; avoid defaulting all functions to the same configuration.
12. For SQS consumers, configure `batchSize` and `functionResponseType: 'ReportBatchItemFailures'` to enable partial batch failure handling.
13. Use Lambda Layers for large shared dependencies (Prisma client, Playwright) to avoid bundle size limits and speed up deploys.

**When handling errors in Lambda**

1. For API Gateway handlers, always return a structured JSON error response — never let unhandled exceptions produce an empty or malformed response.
2. For SQS/SNS/S3 handlers, throw on unrecoverable errors to route messages to the DLQ; catch and log transient errors with retry logic.
3. Log structured JSON to stdout (CloudWatch picks it up automatically); include `requestId`, `functionName`, and relevant context in every log entry.

**When configuring Serverless Framework**

1. Keep `serverless.ts` typed using `import type { AWS } from '@serverless/typescript'`.
2. Organize functions by domain — each function in its own file exported and imported into the main config.
3. Use `${ssm:/path/to/param}` or Secrets Manager references for sensitive values; never commit plaintext secrets.
4. Test locally using `serverless offline` or invoke individual handlers directly with `serverless invoke local`.

## Error Handling

1. When cold starts are unacceptably slow, audit bundle size (`serverless package` + inspect `.serverless/*.zip`), externalize heavy dependencies to a Layer, and verify `esbuild` tree-shaking is active.
2. When SQS messages are not being processed, verify the IAM role has `sqs:ReceiveMessage`, `sqs:DeleteMessage`, and `sqs:GetQueueAttributes` on the queue ARN.
3. When environment variables are undefined at runtime, check that the variable is declared under the correct scope in `serverless.ts` (provider vs. function level) and that the stage is correct.
