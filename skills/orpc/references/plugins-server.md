---
name: plugins-server
description: oRPC server-side plugins — CORS, batch requests, body limit, CSRF, compression, strict GET, rethrow
---

# Server Plugins

Plugins extend handler functionality. All plugins work with RPCHandler, OpenAPIHandler, and custom handlers.

## CORS Plugin

```ts
import { CORSPlugin } from '@orpc/server/plugins'

new CORSPlugin({
  origin: (origin) => origin,
  allowMethods: ['GET', 'HEAD', 'PUT', 'POST', 'DELETE', 'PATCH'],
  exposeHeaders: ['Content-Disposition'],  // needed for OpenAPILink file responses
})
```

## Batch Handler Plugin

Combine multiple requests/responses into a single batch:

```ts
import { BatchHandlerPlugin } from '@orpc/server/plugins'

new BatchHandlerPlugin({
  headers: responses => ({ 'some-header': 'value' }),
})
```

Uses its own batching protocol. Streaming mode sends responses as they arrive; buffered mode collects all first.

**Limitations:** No AsyncIteratorObject or File/Blob in responses.

## Body Limit Plugin

```ts
import { BodyLimitPlugin } from '@orpc/server/plugins'
new BodyLimitPlugin({ maxBodySize: 1024 * 1024 })  // 1MB
```

## Simple CSRF Protection Plugin

```ts
import { SimpleCsrfProtectionPlugin } from '@orpc/server/plugins'
new SimpleCsrfProtectionPlugin({
  headerName: 'x-csrf-token',  // default
})
```

## Strict Get Method Plugin

Blocks GET requests except for explicitly allowed procedures. Enabled by default in RPCHandler.

```ts
import { StrictGetMethodPlugin } from '@orpc/server/plugins'
new StrictGetMethodPlugin()
```

## Compression Plugin

```ts
import { CompressionPlugin } from '@orpc/server/plugins'
new CompressionPlugin()
```

## Rethrow Handler Plugin

Re-throws special errors (like Next.js `redirect`) instead of converting them:

```ts
import { RethrowHandlerPlugin } from '@orpc/server/plugins'
new RethrowHandlerPlugin({
  rethrow: (error) => isRedirectError(error) || isNotFoundError(error),
})
```

## Response Headers Plugin

Inject custom response headers via context:

```ts
import { ResponseHeadersPlugin, ResponseHeadersPluginContext } from '@orpc/server/plugins'

const base = os.$context<ResponseHeadersPluginContext>()

// In middleware:
context.resHeaders?.set('x-custom', 'value')
```

## Request Headers Plugin

Access request headers in context:

```ts
import { RequestHeadersPlugin, RequestHeadersPluginContext } from '@orpc/server/plugins'
// Adds reqHeaders to context
```

## Key Points

- Plugins are collections of interceptors with an `init` method
- Use `order` property to control plugin loading order (lower loads later)
- All handler plugins implement `StandardHandlerPlugin<T>`
- CORSPlugin is essential for cross-origin clients

<!--
Source references:
- https://orpc.unnoq.com/docs/plugins/cors
- https://orpc.unnoq.com/docs/plugins/batch-requests
- https://orpc.unnoq.com/docs/plugins/body-limit
- https://orpc.unnoq.com/docs/plugins/simple-csrf-protection
- https://orpc.unnoq.com/docs/plugins/strict-get-method
- https://orpc.unnoq.com/docs/plugins/compression
- https://orpc.unnoq.com/docs/plugins/rethrow-handler
- https://orpc.unnoq.com/docs/plugins/response-headers
- https://orpc.unnoq.com/docs/plugins/request-headers
-->
