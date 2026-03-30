---
name: core-context
description: oRPC type-safe dependency injection with initial and execution context patterns
---

# Context

oRPC provides type-safe dependency injection through two context types:

- **Initial Context:** Provided explicitly when invoking a procedure
- **Execution Context:** Generated during execution by middleware

## Initial Context

Define required dependencies (usually environment-specific):

```ts
const base = os.$context<{ headers: Headers, env: { DB_URL: string } }>()

const getting = base.handler(async ({ context }) => {
  console.log(context.env)
})
```

Pass initial context when handling requests:

```ts
const handler = new RPCHandler(router)

handler.handle(request, {
  context: {
    headers: request.headers,
    env: { DB_URL: '***' }
  }
})
```

## Execution Context

Computed during the lifecycle via middleware — no manual passing needed:

```ts
const base = os.use(async ({ next }) => next({
  context: {
    headers: await headers(),
    cookies: await cookies(),
  },
}))

// No context needed when handling:
handler.handle(request)
```

## Combining Both

Use initial context for environment values, execution context for runtime data:

```ts
const base = os.$context<{ headers: Headers, env: { DB_URL: string } }>()

const requireAuth = base.middleware(async ({ context, next }) => {
  const user = parseJWT(context.headers.get('authorization')?.split(' ')[1])
  if (user) return next({ context: { user } })
  throw new ORPCError('UNAUTHORIZED')
})

const dbProvider = base.middleware(async ({ context, next }) => {
  const client = new Client(context.env.DB_URL)
  try {
    await client.connect()
    return next({ context: { db: client } })
  } finally {
    await client.disconnect()
  }
})

const getting = base
  .use(dbProvider)
  .use(requireAuth)
  .handler(async ({ context }) => {
    // context.db and context.user are both available and typed
  })
```

## Key Points

- Initial context must be provided when calling a procedure or handling requests
- Execution context is injected by middleware — callers don't need to pass it
- Combine both for maximum flexibility: env config via initial, runtime data via execution
- Context is type-safe throughout the chain

<!--
Source references:
- https://orpc.unnoq.com/docs/context
-->
