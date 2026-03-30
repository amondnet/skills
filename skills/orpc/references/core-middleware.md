---
name: core-middleware
description: oRPC middleware for context injection, guards, input/output interception, and built-in lifecycle hooks
---

# Middleware

Middleware intercepts handler execution, injects/guards context, and modifies input/output.

## Defining Middleware

```ts
const authMiddleware = os
  .$context<{ something?: string }>()  // dependent context requirement
  .middleware(async ({ context, next }) => {
    const result = await next({
      context: { user: { id: 1, name: 'John' } }  // inject context
    })
    return result
  })
```

## Inline Middleware

```ts
const example = os
  .use(async ({ context, next }) => {
    // logic before handler
    return next()
  })
  .handler(async ({ context }) => { /* ... */ })
```

## Context Injection & Guards

```ts
const setting = os
  .use(async ({ context, next }) => {
    return next({ context: { auth: await auth() } })  // inject
  })
  .use(async ({ context, next }) => {
    if (!context.auth) throw new ORPCError('UNAUTHORIZED')  // guard
    return next({ context: { auth: context.auth } })
  })
```

Context passed to `next` is merged with existing context.

## Middleware Input

Access procedure input for permission checks:

```ts
const canUpdate = os.middleware(async ({ context, next }, input: number) => {
  // permission check
  return next()
})

// Direct use when input matches
os.input(z.number()).use(canUpdate)

// Map input when shapes differ
os.input(z.object({ id: z.number() })).use(canUpdate, input => input.id)
```

Use `.mapInput` to transform middleware input shape:

```ts
const mappedCanUpdate = canUpdate.mapInput((input: { id: number }) => input.id)
```

## Middleware Output

Modify handler output (e.g., caching):

```ts
const cacheMid = os.middleware(async ({ context, next, path }, input, output) => {
  const cacheKey = path.join('/') + JSON.stringify(input)
  if (db.has(cacheKey)) return output(db.get(cacheKey))
  const result = await next({})
  db.set(cacheKey, result.output)
  return result
})
```

## Concatenation

Combine multiple middlewares:

```ts
const combined = aMiddleware
  .concat(os.middleware(async ({ next }) => next()))
  .concat(anotherMiddleware)
```

## Built-in Lifecycle Middlewares

```ts
import { onError, onFinish, onStart, onSuccess } from '@orpc/server'

const ping = os
  .use(onStart(() => { /* before handler */ }))
  .use(onSuccess(() => { /* on success */ }))
  .use(onError(() => { /* on failure */ }))
  .use(onFinish(() => { /* after handler */ }))
  .handler(async ({ context }) => { /* ... */ })
```

## Key Points

- Use `.$context<T>()` to declare dependent context requirements
- `next({ context })` merges new context with existing
- Use `.mapInput` to adapt middleware to different input shapes
- Built-in `onStart`, `onSuccess`, `onError`, `onFinish` simplify lifecycle hooks

<!--
Source references:
- https://orpc.unnoq.com/docs/middleware
-->
