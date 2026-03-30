---
name: core-error-handling
description: oRPC error handling with ORPCError class, type-safe errors, and HTTP status code mappings
---

# Error Handling

oRPC supports both throwing standard errors and type-safe predefined error types.

## Normal Approach

Use `ORPCError` for consistent error codes and data:

```ts
import { ORPCError } from '@orpc/server'

throw new ORPCError('NOT_FOUND')
throw new ORPCError('RATE_LIMITED', {
  message: 'You are being rate limited',
  data: { retryAfter: 60 }
})
// Regular errors become INTERNAL_SERVER_ERROR
throw new Error('Something went wrong')
```

**Warning:** `ORPCError.data` is sent to the client — never include sensitive information.

## Type-Safe Error Handling

Define error types with `.errors` for client-inferred error structures:

```ts
const base = os.errors({
  RATE_LIMITED: {
    data: z.object({ retryAfter: z.number() }),
  },
  UNAUTHORIZED: {},
})

const rateLimit = base.middleware(async ({ next, errors }) => {
  throw errors.RATE_LIMITED({
    message: 'You are being rate limited',
    data: { retryAfter: 60 }
  })
})

const example = base
  .use(rateLimit)
  .errors({ NOT_FOUND: { message: 'The resource was not found' } })
  .handler(async ({ input, errors }) => {
    throw errors.NOT_FOUND()
  })
```

## Client-Side Error Handling

```ts
import { isDefinedError, safe } from '@orpc/client'

const [error, data, isDefined] = await safe(doSomething({ id: '123' }))

if (isDefinedError(error)) {
  // Handle known typed error
  console.log(error.data.retryAfter)
} else if (error) {
  // Handle unknown error
} else {
  // Handle success
}
```

Create a safe client wrapper:

```ts
const safeClient = createSafeClient(client)
const [error, data] = await safeClient.doSomething({ id: '123' })
```

## Default Error Code to HTTP Status Mappings

| Error Code | Status | Error Code | Status |
|---|---|---|---|
| BAD_REQUEST | 400 | TOO_MANY_REQUESTS | 429 |
| UNAUTHORIZED | 401 | INTERNAL_SERVER_ERROR | 500 |
| FORBIDDEN | 403 | NOT_IMPLEMENTED | 501 |
| NOT_FOUND | 404 | BAD_GATEWAY | 502 |
| CONFLICT | 409 | SERVICE_UNAVAILABLE | 503 |
| UNPROCESSABLE_CONTENT | 422 | GATEWAY_TIMEOUT | 504 |

Custom status and message can be set per error.

## Key Points

- Both approaches (normal + type-safe) can be combined
- `ORPCError` with matching code/status/data is treated as type-safe
- Never put sensitive data in `ORPCError.data`
- Use `safe()` and `isDefinedError()` on the client for type-safe error handling
- `createSafeClient()` wraps all calls with `safe` automatically

<!--
Source references:
- https://orpc.unnoq.com/docs/error-handling
- https://orpc.unnoq.com/docs/client/error-handling
- https://orpc.unnoq.com/docs/openapi/error-handling
-->
