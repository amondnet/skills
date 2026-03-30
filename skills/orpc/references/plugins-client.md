---
name: plugins-client
description: oRPC client-side and contract plugins — retry, batch, dedupe, retry-after, request/response validation
---

# Client Plugins

Plugins that extend client link behavior. Work with RPCLink, OpenAPILink, and custom links.

## Client Retry Plugin

Enables retrying client calls on errors:

```ts
import { RPCLink } from '@orpc/client/fetch'
import { ClientRetryPlugin, ClientRetryPluginContext } from '@orpc/client/plugins'

interface ORPCClientContext extends ClientRetryPluginContext {}

const link = new RPCLink<ORPCClientContext>({
  url: 'http://localhost:3000/rpc',
  plugins: [
    new ClientRetryPlugin({
      default: {
        retry: ({ path }) => path.join('.') === 'planet.list' ? 2 : 0,
      },
    }),
  ],
})
```

Per-call configuration:

```ts
const result = await client.planet.list({ limit: 10 }, {
  context: {
    retry: 3,
    retryDelay: 2000,
    shouldRetry: (options) => true,
    onRetry: (options) => (isSuccess) => { /* ... */ },
  },
})
```

For EventSource-like behavior with Event Iterator:

```ts
const streaming = await client.streaming('input', {
  context: { retry: Number.POSITIVE_INFINITY },
})
```

## Batch Link Plugin

Combine multiple requests into a single batch:

```ts
import { BatchLinkPlugin } from '@orpc/client/plugins'

const link = new RPCLink({
  url: 'https://api.example.com/rpc',
  plugins: [
    new BatchLinkPlugin({
      mode: 'streaming',  // or 'buffered' for serverless
      groups: [
        { condition: () => true, context: {} }
      ],
      exclude: ({ path }) => path.join('/') === 'planets/subscribe',
    }),
  ],
})
```

Requires `BatchHandlerPlugin` on the server. Groups enable batching by context (e.g., cache control).

## Dedupe Requests Plugin

Prevents duplicate concurrent requests:

```ts
import { DedupeRequestsPlugin } from '@orpc/client/plugins'

const link = new RPCLink({
  plugins: [
    new DedupeRequestsPlugin({
      filter: ({ request }) => request.method === 'GET',
      groups: [{ condition: () => true, context: {} }],
    }),
  ],
})
```

By default, only GET requests are deduplicated. Expanding to all requests also prevents double-click mutation issues.

## Retry After Plugin

Automatically retries requests based on server `Retry-After` headers:

```ts
import { RetryAfterPlugin } from '@orpc/client/plugins'

const link = new RPCLink({
  url: 'http://localhost:3000/rpc',
  plugins: [
    new RetryAfterPlugin({
      condition: (response) => response.status === 429 || response.status === 503,
      maxAttempts: 5,
      timeout: 5 * 60 * 1000,
    }),
  ],
})
```

## Request/Response Validation Plugins

Validate requests/responses against a contract schema on the client side. From `@orpc/contract/plugins`:

```ts
import { RequestValidationPlugin } from '@orpc/contract/plugins'
import { ResponseValidationPlugin } from '@orpc/contract/plugins'

const link = new RPCLink({
  url: 'http://localhost:3000/rpc',
  plugins: [
    new RequestValidationPlugin(contract),
    new ResponseValidationPlugin(contract),
  ],
})
```

Best suited for contract-first development. Minified contracts are not supported.

## Key Points

- Retry is disabled by default — set `retry` count to enable
- Default retry delay is `(o) => o.lastEventRetry ?? 2000`
- Batch plugin has streaming (default) and buffered modes
- Requests in the same group are considered for batching/deduplication
- Each group's `context` is used for the rest of the request lifecycle

<!--
Source references:
- https://orpc.unnoq.com/docs/plugins/client-retry
- https://orpc.unnoq.com/docs/plugins/batch-requests
- https://orpc.unnoq.com/docs/plugins/dedupe-requests
- https://orpc.unnoq.com/docs/plugins/retry-after
- https://orpc.unnoq.com/docs/plugins/request-validation
- https://orpc.unnoq.com/docs/plugins/response-validation
-->
