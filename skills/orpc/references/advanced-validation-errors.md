---
name: advanced-validation-errors
description: Customizing oRPC validation error responses with interceptors and type-safe error codes
---

# Validation Errors

oRPC provides built-in validation errors. Customize them via client interceptors or middleware.

## Customizing with Client Interceptors (Preferred)

Client interceptors run before error validation, ensuring custom errors are properly validated:

```ts
import { onError, ORPCError, ValidationError } from '@orpc/server'

const handler = new RPCHandler(router, {
  clientInterceptors: [
    onError((error) => {
      if (
        error instanceof ORPCError
        && error.code === 'BAD_REQUEST'
        && error.cause instanceof ValidationError
      ) {
        const zodError = new z.ZodError(error.cause.issues as z.core.$ZodIssue[])
        throw new ORPCError('INPUT_VALIDATION_FAILED', {
          status: 422,
          message: z.prettifyError(zodError),
          data: z.flattenError(zodError),
          cause: error.cause,
        })
      }

      if (
        error instanceof ORPCError
        && error.code === 'INTERNAL_SERVER_ERROR'
        && error.cause instanceof ValidationError
      ) {
        throw new ORPCError('OUTPUT_VALIDATION_FAILED', { cause: error.cause })
      }
    }),
  ],
})
```

## Customizing with Middleware

```ts
const base = os.use(onError((error) => {
  if (error instanceof ORPCError && error.code === 'BAD_REQUEST'
      && error.cause instanceof ValidationError) {
    const zodError = new z.ZodError(error.cause.issues as z.core.$ZodIssue[])
    throw new ORPCError('INPUT_VALIDATION_FAILED', {
      status: 422,
      message: z.prettifyError(zodError),
      data: z.flattenError(zodError),
      cause: error.cause,
    })
  }
}))
```

## Type-Safe Validation Errors

Combine with `.errors` for full type safety:

```ts
const base = os.errors({
  INPUT_VALIDATION_FAILED: {
    status: 422,
    data: z.object({
      formErrors: z.array(z.string()),
      fieldErrors: z.record(z.string(), z.array(z.string()).optional()),
    }),
  },
})
```

When the thrown `ORPCError` code/status/data matches `.errors`, oRPC treats it as type-safe.

## Key Points

- Input validation errors have code `BAD_REQUEST` with `ValidationError` cause
- Output validation errors have code `INTERNAL_SERVER_ERROR` with `ValidationError` cause
- `ValidationError.issues` follows Standard Schema issue format
- Client interceptors are preferred — they run before error validation
- Middleware before `.input`/`.output` catches validation errors by default

<!--
Source references:
- https://orpc.unnoq.com/docs/advanced/validation-errors
-->
