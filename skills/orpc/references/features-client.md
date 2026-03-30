---
name: features-client
description: oRPC client creation — RPCLink, OpenAPILink, DynamicLink, server-side and client-side clients
---

# Client

Call oRPC procedures remotely as local functions or directly on the server.

## Client-Side Client

```ts
import { createORPCClient } from '@orpc/client'
import { RPCLink } from '@orpc/client/fetch'
import { RouterClient } from '@orpc/server'

const link = new RPCLink({
  url: 'http://localhost:3000/rpc',
  headers: () => ({ authorization: 'Bearer token' }),
})

const client: RouterClient<typeof router> = createORPCClient(link)

// Call procedures like functions
const planet = await client.planet.find({ id: 1 })
```

## Server-Side Client

Call procedures directly without HTTP:

```ts
// Using .callable()
const result = await getProcedure({ id: '123' })

// Using call utility
import { call } from '@orpc/server'
const result = await call(getProcedure, { id: '123' }, { context: {} })

// Router client
import { createRouterClient } from '@orpc/server'
const client = createRouterClient({ ping, pong }, { context: {} })
```

## RPCLink

Communicates with RPCHandler over oRPC's proprietary protocol:

```ts
const link = new RPCLink({
  url: 'http://localhost:3000/rpc',
  method: ({ context }, path) => {
    if (path.at(-1)?.match(/^(?:get|find|list|search)(?:[A-Z].*)?$/)) return 'GET'
    return 'POST'
  },
  fetch: (request, init) => globalThis.fetch(request, {
    ...init, credentials: 'include'
  }),
})
```

## OpenAPILink

Communicates with OpenAPIHandler over RESTful API:

```ts
import { OpenAPILink } from '@orpc/openapi-client/fetch'
import type { JsonifiedClient } from '@orpc/openapi-client'

const link = new OpenAPILink(contract, { url: 'http://localhost:3000/api' })
const client: JsonifiedClient<ContractRouterClient<typeof contract>> = createORPCClient(link)
```

Requires `JsonifiedClient` wrapper due to JSON limitations.

## DynamicLink

Dynamically choose links based on context:

```ts
import { DynamicLink } from '@orpc/client'

const link = new DynamicLink<ClientContext>((options, path, input) => {
  return options.context?.cache ? cacheLink : noCacheLink
})
```

## Client Context

Pass extra information when calling procedures:

```ts
const result = await client.planet.list(
  { limit: 10 },
  { context: { something: 'value' } }
)
```

Required properties in `ClientContext` are enforced at call sites.

## Merge Clients

```ts
export const orpc = {
  a: createORPCClient(linkA),
  b: createORPCClient(linkB),
}
```

## Type Inference Utilities

```ts
import type { InferClientInputs, InferClientOutputs, InferClientErrors } from '@orpc/client'

type Inputs = InferClientInputs<typeof client>
type Outputs = InferClientOutputs<typeof client>
type Errors = InferClientErrors<typeof client>
```

## Key Points

- RPCLink for oRPC protocol (full type fidelity); OpenAPILink for REST
- Server-side client has zero network overhead — ideal for SSR and testing
- DynamicLink enables runtime link selection
- Client context can modify link behavior (headers, cache, method)
- Lazy URL function prevents runtime API errors in SSR

<!--
Source references:
- https://orpc.unnoq.com/docs/client/client-side
- https://orpc.unnoq.com/docs/client/server-side
- https://orpc.unnoq.com/docs/client/rpc-link
- https://orpc.unnoq.com/docs/client/dynamic-link
- https://orpc.unnoq.com/docs/openapi/client/openapi-link
-->
