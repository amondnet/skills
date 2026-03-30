---
name: features-handler
description: oRPC RPCHandler and OpenAPIHandler setup, supported data types, and request handling
---

# Handlers

oRPC provides two handler types for serving procedures over HTTP.

## RPCHandler

Uses oRPC's proprietary RPC protocol — efficient but not human-readable or OpenAPI-compatible.

```ts
import { RPCHandler } from '@orpc/server/fetch'  // or '@orpc/server/node'
import { CORSPlugin } from '@orpc/server/plugins'

const handler = new RPCHandler(router, {
  plugins: [new CORSPlugin()],
  interceptors: [onError((error) => console.error(error))],
})

const { matched, response } = await handler.handle(request, {
  prefix: '/rpc',
  context: {}
})
```

Designed exclusively for RPCLink — do not send manual requests.

### Supported Native Types

string, number (including NaN), boolean, null, undefined, Date, BigInt, RegExp, URL, Record, Array, Set, Map, Blob, File, AsyncIteratorObject (root level only).

## OpenAPIHandler

Serves RESTful endpoints following the OpenAPI specification.

```ts
import { OpenAPIHandler } from '@orpc/openapi/fetch'

const handler = new OpenAPIHandler(router, {
  plugins: [new CORSPlugin()],
})

const { matched, response } = await handler.handle(request, {
  prefix: '/api',
  context: {}
})
```

### Data Type Serialization

Same types as RPCHandler, but with JSON-safe conversions: NaN → null, Date → ISO string, BigInt → string, RegExp → string, URL → string, Set → array, Map → array.

## Filtering Procedures

Exclude procedures from matching:

```ts
const handler = new RPCHandler(router, {
  filter: ({ contract, path }) => !contract['~orpc'].route.tags?.includes('internal'),
})
```

## Running Both Handlers

You can run both RPCHandler and OpenAPIHandler simultaneously:

```ts
export default async function fetch(request: Request) {
  const url = new URL(request.url)

  if (url.pathname.startsWith('/rpc')) {
    const { response } = await rpcHandler.handle(request, { prefix: '/rpc', context: {} })
    if (response) return response
  }

  if (url.pathname.startsWith('/api')) {
    const { response } = await openAPIHandler.handle(request, { prefix: '/api', context: {} })
    if (response) return response
  }

  return new Response('Not Found', { status: 404 })
}
```

## Key Points

- RPCHandler for RPCLink (full type fidelity), OpenAPIHandler for REST clients
- Both support fetch and Node.js adapters
- Use `prefix` to mount at a specific path
- Default plugins (like StrictGetMethodPlugin) are enabled for security
- Interceptors can be added at multiple lifecycle stages

<!--
Source references:
- https://orpc.unnoq.com/docs/rpc-handler
- https://orpc.unnoq.com/docs/openapi/openapi-handler
-->
