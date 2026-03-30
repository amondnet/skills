---
name: features-contract-first
description: Contract-first development with oRPC — define, implement, and convert contracts
---

# Contract-First Development

Define the API contract before writing implementation code.

## Define Contract

```ts
import { oc } from '@orpc/contract'  // @orpc/contract package

const listPlanetContract = oc
  .input(z.object({
    limit: z.number().int().min(1).max(100).optional(),
    cursor: z.number().int().min(0).default(0),
  }))
  .output(z.array(PlanetSchema))

const contract = {
  planet: {
    list: listPlanetContract,
    find: findPlanetContract,
    create: createPlanetContract,
  },
}
```

Type inference utilities:

```ts
import type { InferContractRouterInputs, InferContractRouterOutputs } from '@orpc/contract'

type Inputs = InferContractRouterInputs<typeof contract>
type Outputs = InferContractRouterOutputs<typeof contract>
```

## Implement Contract

The `implement` function converts a contract into an implementer with full type safety:

```ts
import { implement } from '@orpc/server'

const os = implement(contract)  // replaces os from @orpc/server

const listPlanet = os.planet.list
  .handler(({ input }) => {
    return []
  })

// Build router with full contract enforcement
const router = os.router({
  planet: { list: listPlanet, find: findPlanet, create: createPlanet },
})
```

## Router to Contract

A normal router works as a contract if it has no lazy routes.

### Unlazy the Router

```ts
import { unlazyRouter } from '@orpc/server'
const resolvedRouter = await unlazyRouter(router)
```

### Minify & Export for Client

Prevent leaking server logic to the client:

```ts
import { minifyContractRouter } from '@orpc/contract'
const minified = minifyContractRouter(router)
fs.writeFileSync('./contract.json', JSON.stringify(minified))
```

Import on client:

```ts
import contract from './contract.json'
const link = new OpenAPILink(contract as typeof router, { url: '...' })
```

## Key Points

- Use `oc` from `@orpc/contract` for contract definitions
- Use `implement(contract)` as a drop-in replacement for `os`
- `os.router()` ensures full contract enforcement at the router level
- `minifyContractRouter` strips business logic for safe client-side import
- `implement` also useful for creating mock/fake implementations in tests

<!--
Source references:
- https://orpc.unnoq.com/docs/contract-first/define-contract
- https://orpc.unnoq.com/docs/contract-first/implement-contract
- https://orpc.unnoq.com/docs/contract-first/router-to-contract
-->
