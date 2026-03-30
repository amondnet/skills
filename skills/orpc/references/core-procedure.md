---
name: core-procedure
description: oRPC procedure definition with input/output validation, middleware chaining, and handler patterns
---

# Procedure

A procedure is a function with built-in input/output validation, middleware, and dependency injection.

## Defining a Procedure

```ts
import { os } from '@orpc/server'

const example = os
  .use(aMiddleware)                              // Apply middleware
  .input(z.object({ name: z.string() }))         // Input validation
  .use(aMiddlewareWithInput, input => input.name) // Middleware with typed input
  .output(z.object({ id: z.number() }))           // Output validation
  .handler(async ({ input, context }) => {         // Execution logic
    return { id: 1 }
  })
  .callable()    // Make callable like a regular function
  .actionable()  // Server Action compatibility
```

Only `.handler` is required. All other chains are optional.

## Input/Output Validation

Supports Zod, Valibot, ArkType, and any Standard Schema library.

**Tip:** Explicitly specifying `.output` or handler return type improves TypeScript performance.

### `type` Utility

For simple cases without external libraries:

```ts
import { os, type } from '@orpc/server'

const example = os
  .input(type<{ value: number }>())
  .output(type<{ value: number }, number>(({ value }) => value))
  .handler(async ({ input }) => input)
```

## Initial Configuration

Use `.$input` to set a redefinable initial input schema:

```ts
const base = os.$input(z.void())       // enforce void input by default
const base = os.$input<Schema<void, unknown>>()
```

## Reusability

Each builder modification creates a new instance — safe for branching:

```ts
const pub = os.use(logMiddleware)
const authed = pub.use(authMiddleware)

const pubExample = pub.handler(async ({ context }) => { /* ... */ })
const authedExample = pubExample.use(authMiddleware)
```

## Key Points

- Every procedure is also a router — router utilities work on procedures
- `.handler` is the only required step
- Validation runs automatically before/after the handler
- Use `.callable()` for direct server-side invocation
- Use `.actionable()` for React Server Action compatibility

<!--
Source references:
- https://orpc.unnoq.com/docs/procedure
-->
