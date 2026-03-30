---
name: best-practices-patterns
description: oRPC best practices — middleware deduplication, SSR optimization, testing, and metadata
---

# Best Practices

## Dedupe Middleware

When procedures call each other or routers extend procedures, middleware may execute multiple times.

### Manual Deduplication Pattern

Use context to track execution:

```ts
const dbProvider = os
  .$context<{ db?: Awaited<ReturnType<typeof connectDb>> }>()
  .middleware(async ({ context, next }) => {
    const db = context.db ?? await connectDb()  // skip if already connected
    return next({ context: { db } })
  })
```

### Built-in Deduplication

oRPC auto-dedupes when router middlewares are a subset of leading procedure middlewares in the same order:

```ts
const router = os.use(logging).use(dbProvider).router({
  // ✅ Deduped: os.use(logging).use(dbProvider).use(auth).handler(...)
  // ⛔ Not deduped: os.use(dbProvider).use(logging).handler(...)  (different order)
})
```

Disable with: `os.$config({ dedupeLeadingMiddlewares: false })`

## Optimize SSR

Avoid the server calling its own API over HTTP during SSR.

### Direct Server-Side Client via globalThis

```ts
// lib/orpc.server.ts
import 'server-only'
globalThis.$client = createRouterClient(router, {
  context: async () => ({ headers: await headers() }),
})

// lib/orpc.ts (shared)
export const client: RouterClient<typeof router> =
  globalThis.$client ?? createORPCClient(link)
```

Import `lib/orpc.server.ts` in `instrumentation.ts` and `app/layout.tsx` (Next.js).

### Alternative: Fetch Adapter Approach

Route client through the handler's fetch function for plugin support (e.g., DedupeRequestsPlugin):

```ts
const link = new RPCLink({
  url: 'http://placeholder',
  fetch: async (request) => {
    const { response } = await handler.handle(request, { context: { headers: await headers() } })
    return response ?? new Response('Not Found', { status: 404 })
  },
})
```

## Testing & Mocking

### Testing with Server-Side Client

```ts
import { call } from '@orpc/server'

it('works', async () => {
  await expect(
    call(router.planet.list, { page: 1, size: 10 })
  ).resolves.toEqual([...])
})
```

### Mocking with Implementer

```ts
import { implement } from '@orpc/server'
const fakeListPlanet = implement(router.planet.list).handler(() => [])
```

## Metadata

Attach key-value pairs to procedures for cross-cutting behavior:

```ts
interface ORPCMetadata { cache?: boolean }

const base = os
  .$meta<ORPCMetadata>({})
  .use(async ({ procedure, next, path }, input, output) => {
    if (!procedure['~orpc'].meta.cache) return await next()
    // caching logic
  })

const example = base.meta({ cache: true }).handler(() => { /* ... */ })
```

`.meta` can be called multiple times; each call spread-merges with existing metadata.

## Key Points

- Always use context-based deduplication for expensive middleware (DB, auth)
- Built-in dedup only works for leading middleware in the same order
- SSR optimization eliminates redundant HTTP calls for server rendering
- `implement()` creates mock implementations from contracts or procedures
- Metadata enables cross-cutting concerns without coupling to specific middleware

<!--
Source references:
- https://orpc.unnoq.com/docs/best-practices/dedupe-middleware
- https://orpc.unnoq.com/docs/best-practices/optimize-ssr
- https://orpc.unnoq.com/docs/advanced/testing-mocking
- https://orpc.unnoq.com/docs/metadata
-->
