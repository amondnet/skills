---
name: orpc
description: Type-safe API development with oRPC — end-to-end typesafe procedures, OpenAPI support, streaming, and multi-framework integrations
metadata:
  author: Minsu Lee
  version: "2026.3.30"
  source: Generated from https://github.com/middleapi/orpc, scripts located at https://github.com/amondnet/skills
---

> The skill is based on oRPC v1.13.13, generated at 2026-03-30.

oRPC is a TypeScript-first RPC framework for building end-to-end typesafe APIs. It provides procedure-based API definitions with input/output validation (Zod, Valibot, ArkType), middleware chains, type-safe error handling, OpenAPI 3.1.1 spec generation, streaming via Event Iterators (SSE), contract-first development, and integrations with TanStack Query, AI SDK, React Server Actions, and more.

## Core References

| Topic | Description | Reference |
|-------|-------------|-----------|
| Procedure | Defining procedures with validation, middleware, and handlers | [core-procedure](references/core-procedure.md) |
| Router | Router composition, lazy loading, and type inference | [core-router](references/core-router.md) |
| Middleware | Context injection, guards, input/output interception, lifecycle hooks | [core-middleware](references/core-middleware.md) |
| Context | Type-safe dependency injection with initial and execution context | [core-context](references/core-context.md) |
| Error Handling | ORPCError, type-safe errors, HTTP status mappings, client error handling | [core-error-handling](references/core-error-handling.md) |

## Features

### Handlers & Transport

| Topic | Description | Reference |
|-------|-------------|-----------|
| Handlers | RPCHandler and OpenAPIHandler setup, data types, lifecycle | [features-handler](references/features-handler.md) |
| Client | RPCLink, OpenAPILink, DynamicLink, server/client-side clients | [features-client](references/features-client.md) |
| OpenAPI | Spec generation, routing, input/output structure, operation metadata | [features-openapi](references/features-openapi.md) |

### Real-time & Streaming

| Topic | Description | Reference |
|-------|-------------|-----------|
| Event Iterator | Streaming/SSE with async generators, EventPublisher, resume support | [features-event-iterator](references/features-event-iterator.md) |

### Framework Integration

| Topic | Description | Reference |
|-------|-------------|-----------|
| Server Action | React Server Actions with useServerAction, optimistic updates, forms | [features-server-action](references/features-server-action.md) |
| Contract First | Define, implement, and convert contracts for API-first development | [features-contract-first](references/features-contract-first.md) |
| TanStack Query | Query/mutation options, hydration, streaming queries, key management | [features-tanstack-query](references/features-tanstack-query.md) |

## Plugins

| Topic | Description | Reference |
|-------|-------------|-----------|
| Server Plugins | CORS, batch, body limit, CSRF, compression, strict GET, rethrow, response/request headers | [plugins-server](references/plugins-server.md) |
| Client Plugins | Retry, batch, dedupe, retry-after, request/response validation | [plugins-client](references/plugins-client.md) |

## Integrations

| Topic | Description | Reference |
|-------|-------------|-----------|
| AI SDK | Vercel AI SDK streaming, tool creation, chat patterns | [integrations-ai-sdk](references/integrations-ai-sdk.md) |
| Observability | OpenTelemetry instrumentation, Pino logging | [integrations-observability](references/integrations-observability.md) |

## Best Practices & Advanced

| Topic | Description | Reference |
|-------|-------------|-----------|
| Best Practices | Middleware dedup, SSR optimization, testing/mocking, metadata | [best-practices-patterns](references/best-practices-patterns.md) |
| Custom Plugins | Building handler and link plugins with interceptors | [advanced-custom-plugins](references/advanced-custom-plugins.md) |
| Validation Errors | Customizing validation error responses | [advanced-validation-errors](references/advanced-validation-errors.md) |
| Helpers | Rate limiting, publisher, cookie, encryption, signing, form data | [advanced-helpers](references/advanced-helpers.md) |
