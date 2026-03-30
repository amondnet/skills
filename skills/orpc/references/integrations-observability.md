---
name: integrations-observability
description: oRPC OpenTelemetry instrumentation and Pino logging integration
---

# Observability

## OpenTelemetry

oRPC integrates with OpenTelemetry for distributed tracing via `@orpc/otel`.

### Setup

```ts
import { NodeSDK } from '@opentelemetry/sdk-node'
import { ORPCInstrumentation } from '@orpc/otel'

const sdk = new NodeSDK({
  instrumentations: [new ORPCInstrumentation()],
})
sdk.start()
```

Works on both server (NodeSDK) and client (WebTracerProvider).

### Middleware Spans

oRPC auto-creates spans per middleware. Access the active span:

```ts
import { trace } from '@opentelemetry/api'

const someMiddleware = os.middleware(async (ctx, next) => {
  const span = trace.getActiveSpan()
  span?.setAttribute('someAttribute', 'someValue')
  return next()
})

// Name middleware for better span naming
Object.defineProperty(someMiddleware, 'name', { value: 'someName' })
```

### Capture Abort Signals

For streaming/Event Iterator patterns:

```ts
const handler = new RPCHandler(router, {
  interceptors: [
    ({ request, next }) => {
      const span = trace.getActiveSpan()
      request.signal?.addEventListener('abort', () => {
        span?.addEvent('aborted', { reason: String(request.signal?.reason) })
      })
      return next()
    },
  ],
})
```

### Context Propagation

Set up HTTP instrumentation (e.g., `@opentelemetry/instrumentation-http`, `@hono/otel`) for trace context propagation between services.

## Pino Logging

Use `@orpc/pino` for structured logging integration with oRPC procedures.

## Key Points

- `ORPCInstrumentation` auto-instruments both client and server
- Server-only instrumentation is sufficient for most use cases
- Name your middleware functions for readable trace spans
- Capture abort signals for proper streaming operation tracking

<!--
Source references:
- https://orpc.unnoq.com/docs/integrations/opentelemetry
- https://orpc.unnoq.com/docs/integrations/pino
-->
