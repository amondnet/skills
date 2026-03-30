---
name: features-tanstack-query
description: oRPC TanStack Query integration for React/Vue/Solid/Svelte with query options, mutations, and hydration
---

# TanStack Query Integration

Lightweight integration supporting all TanStack Query libraries (React, Vue, Angular, Solid, Svelte).

## Setup

```ts
import { createTanstackQueryUtils } from '@orpc/tanstack-query'

export const orpc = createTanstackQueryUtils(client)
```

Avoid key conflicts with unique base paths:

```ts
const userORPC = createTanstackQueryUtils(userClient, { path: ['user'] })
const postORPC = createTanstackQueryUtils(postClient, { path: ['post'] })
```

## Query Options

```ts
const query = useQuery(orpc.planet.find.queryOptions({
  input: { id: 123 },
  context: { cache: true },
}))
```

## Streamed Query Options (Event Iterator)

Data is an array of events, each appended as it arrives:

```ts
const query = useQuery(orpc.streamed.experimental_streamedOptions({
  input: { id: 123 },
  queryFnOptions: { refetchMode: 'reset', maxChunks: 3 },
  retry: true,
}))
```

## Live Query Options

Data is always the latest event:

```ts
const query = useQuery(orpc.live.experimental_liveOptions({
  input: { id: 123 },
  retry: true,
}))
```

## Infinite Query Options

```ts
const query = useInfiniteQuery(orpc.planet.list.infiniteOptions({
  input: (pageParam: number | undefined) => ({ limit: 10, offset: pageParam }),
  initialPageParam: undefined,
  getNextPageParam: lastPage => lastPage.nextPageParam,
}))
```

## Mutation Options

```ts
const mutation = useMutation(orpc.planet.create.mutationOptions())
mutation.mutate({ name: 'Earth' })
```

## Query/Mutation Keys

```ts
// Partial match — invalidate all planet queries
queryClient.invalidateQueries({ queryKey: orpc.planet.key() })

// Full match — update specific query data
queryClient.setQueryData(
  orpc.planet.find.queryKey({ input: { id: 123 } }),
  (old) => ({ ...old, name: 'Earth' })
)
```

Key helpers: `.key()`, `.queryKey()`, `.streamedKey()`, `.infiniteKey()`, `.mutationKey()`.

## skipToken

Type-safe alternative to `disabled` for conditional queries:

```ts
const query = useQuery(orpc.planet.list.queryOptions({
  input: search ? { search } : skipToken,
}))
```

## Default Options

```ts
const orpc = createTanstackQueryUtils(client, {
  experimental_defaults: {
    planet: {
      find: { queryOptions: { staleTime: 60_000, retry: 3 } },
      create: {
        mutationOptions: {
          onSuccess: (output, input, _, ctx) => {
            ctx.client.invalidateQueries({ queryKey: orpc.planet.key() })
          },
        },
      },
    },
  },
})
```

## Hydration (SSR)

Use `StandardRPCJsonSerializer` for TanStack Query hydration to support all oRPC types:

```ts
import { StandardRPCJsonSerializer } from '@orpc/client/standard'

const serializer = new StandardRPCJsonSerializer()

const queryClient = new QueryClient({
  defaultOptions: {
    queries: {
      queryKeyHashFn: (key) => JSON.stringify(serializer.serialize(key)),
      staleTime: 60_000,
    },
    dehydrate: {
      serializeData: (data) => serializer.serialize(data),
    },
    hydrate: {
      deserializeData: (data) => serializer.deserialize(data.json, data.meta),
    },
  },
})
```

## Error Handling

```ts
const mutation = useMutation(orpc.planet.create.mutationOptions({
  onError: (error) => {
    if (isDefinedError(error)) { /* type-safe error */ }
  },
}))
```

## Key Points

- Client context is excluded from query keys — override manually if needed
- Prefer built-in `retry` option over oRPC Client Retry Plugin with TanStack Query
- `.call` is an alias for directly calling the underlying procedure client
- Reactive options supported in Vue and Solid (functions or computed values)

<!--
Source references:
- https://orpc.unnoq.com/docs/integrations/tanstack-query
-->
