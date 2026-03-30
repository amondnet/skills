---
name: core-router
description: oRPC router composition, lazy loading for code splitting, and type inference utilities
---

# Router

Routers are plain JavaScript objects of procedures, supporting nesting and middleware extension.

## Defining a Router

```ts
const router = {
  ping,
  pong,
  nested: { ping, pong }
}
```

## Extending a Router

Apply middleware to all procedures in a router:

```ts
const router = os.use(requiredAuth).router({
  ping,
  pong,
  nested: { ping, pong }
})
```

**Warning:** Middleware applied at both router and procedure levels may execute multiple times. See dedupe middleware best practices.

## Lazy Router (Code Splitting)

Defer route initialization for better cold start performance:

```ts
const router = {
  ping,
  pong,
  planet: os.lazy(() => import('./planet'))
}
```

The standalone `lazy` helper from `@orpc/server` is faster for type inference and doesn't require matching initial context:

```ts
import { lazy } from '@orpc/server'

const router = {
  planet: lazy(() => import('./planet'))
}
```

## Type Inference Utilities

```ts
import type {
  InferRouterInputs,
  InferRouterOutputs,
  InferRouterInitialContexts,
  InferRouterCurrentContexts,
} from '@orpc/server'

type Inputs = InferRouterInputs<typeof router>
type FindInput = Inputs['planet']['find']

type Outputs = InferRouterOutputs<typeof router>
type FindOutput = Outputs['planet']['find']
```

## Key Points

- Routers are plain objects — compose freely
- Every procedure is also a router
- Lazy routers enable code splitting for cold start optimization
- Use `unlazyRouter()` to resolve lazy routers when needed (e.g., for contracts)

<!--
Source references:
- https://orpc.unnoq.com/docs/router
-->
