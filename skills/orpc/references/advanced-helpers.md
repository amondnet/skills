---
name: advanced-helpers
description: oRPC helper packages — rate limiting, publisher (real-time), cookie, encryption, signing, form data
---

# Helpers

oRPC provides helper packages for common server-side patterns.

## Rate Limiting (`@orpc/experimental-ratelimit`)

Multiple storage backends: Memory, Redis, Upstash, Cloudflare.

```ts
import { MemoryRatelimiter } from '@orpc/experimental-ratelimit/memory'

const limiter = new MemoryRatelimiter({
  maxRequests: 10,
  window: 60000,  // 60 seconds
})
```

### As Middleware

```ts
import { createRatelimitMiddleware } from '@orpc/experimental-ratelimit'

os.use(createRatelimitMiddleware({
  limiter: ({ context }) => context.ratelimiter,
  key: ({ context }, input) => `login:${input.email}`,
}))
```

Auto-deduplicates when same limiter/key used multiple times. Disable with `dedupe: false`.

### Handler Plugin for Headers

```ts
import { RatelimitHandlerPlugin } from '@orpc/experimental-ratelimit'
new RatelimitHandlerPlugin()  // adds RateLimit-* and Retry-After headers
```

### Adapters

| Adapter | Package | Resume |
|---|---|---|
| Memory | `@orpc/experimental-ratelimit/memory` | N/A |
| Redis (ioredis) | `@orpc/experimental-ratelimit/redis` | N/A |
| Upstash | `@orpc/experimental-ratelimit/upstash-ratelimit` | N/A |
| Cloudflare | `@orpc/experimental-ratelimit/cloudflare-ratelimit` | N/A |

## Publisher (`@orpc/experimental-publisher`)

Pub/sub with resume support for real-time features.

```ts
import { MemoryPublisher } from '@orpc/experimental-publisher/memory'

const publisher = new MemoryPublisher<{
  'something-updated': { id: string }
}>({ resumeRetentionSeconds: 120 })

// Subscribe in handler
const live = os.handler(async function* ({ signal, lastEventId }) {
  for await (const payload of publisher.subscribe('event', { signal, lastEventId })) {
    yield payload
  }
})

// Publish from another handler
await publisher.publish('something-updated', { id: '123' })
```

### Adapters

| Adapter | Package | Resume |
|---|---|---|
| Memory | `@orpc/experimental-publisher/memory` | ✅ |
| IORedis | `@orpc/experimental-publisher/ioredis` | ✅ |
| Upstash Redis | `@orpc/experimental-publisher/upstash-redis` | ✅ |
| Cloudflare Durable Objects | `@orpc/experimental-publisher-durable-object` | ✅ |

Resume is disabled by default — enable with `resumeRetentionSeconds`.

## Other Helpers

- **Cookie** (`@orpc/server`): Cookie parsing and serialization
- **Encryption** (`@orpc/server`): Encryption utilities
- **Signing** (`@orpc/server`): Request signing
- **Base64URL** (`@orpc/server`): Base64URL encoding/decoding
- **Form Data** (`@orpc/react`): `parseFormData`, `getIssueMessage` with bracket notation

## Key Points

- Rate limiters support blocking mode (wait for reset instead of rejecting)
- Publisher manages event IDs automatically when resume is enabled
- IORedis publisher requires two separate Redis connections (commander + listener)
- Use `createRatelimitMiddleware` for seamless oRPC integration with auto-dedup
- Combine `RatelimitHandlerPlugin` with `RetryAfterPlugin` for automatic client retries

<!--
Source references:
- https://orpc.unnoq.com/docs/helpers/ratelimit
- https://orpc.unnoq.com/docs/helpers/publisher
- https://orpc.unnoq.com/docs/helpers/cookie
- https://orpc.unnoq.com/docs/helpers/encryption
- https://orpc.unnoq.com/docs/helpers/signing
- https://orpc.unnoq.com/docs/helpers/base64url
- https://orpc.unnoq.com/docs/helpers/form-data
-->
