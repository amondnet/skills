---
name: features-event-iterator
description: oRPC streaming responses and SSE with async generators, EventPublisher, and event metadata
---

# Event Iterator (SSE)

Built-in support for streaming responses, real-time updates, and server-sent events.

## Server-Side

Define streaming handlers with async generators:

```ts
const example = os
  .handler(async function* ({ input, lastEventId }) {
    while (true) {
      yield { message: 'Hello, world!' }
      await new Promise(resolve => setTimeout(resolve, 1000))
    }
  })
```

### Validate Events

```ts
import { eventIterator } from '@orpc/server'

const example = os
  .output(eventIterator(z.object({ message: z.string() })))
  .handler(async function* () {
    yield { message: 'Hello, world!' }
  })
```

### Event Metadata (Last Event ID)

Attach event ID and retry interval for resume support:

```ts
import { withEventMeta } from '@orpc/server'

const example = os
  .handler(async function* ({ input, lastEventId }) {
    if (lastEventId) {
      // Resume streaming from lastEventId
    }
    while (true) {
      yield withEventMeta({ message: 'Hello' }, { id: 'some-id', retry: 10_000 })
      await new Promise(resolve => setTimeout(resolve, 1000))
    }
  })
```

### Stop & Cleanup

Use `return` to end the stream. Use `finally` for cleanup:

```ts
const example = os
  .handler(async function* () {
    try {
      while (true) {
        if (done) return  // signals successful completion
        yield { message: 'Hello' }
      }
    } finally {
      console.log('Cleanup logic here')
    }
  })
```

## EventPublisher (Lightweight)

Synchronous publishing, no resume support:

```ts
import { EventPublisher } from '@orpc/server'

const publisher = new EventPublisher<{
  'something-updated': { id: string }
}>()

// Subscribe
const live = os.handler(async function* ({ signal }) {
  for await (const payload of publisher.subscribe('something-updated', { signal })) {
    yield payload
  }
})

// Publish
const update = os.handler(({ input }) => {
  publisher.publish('something-updated', { id: input.id })
})
```

## Client-Side Consumption

```ts
const iterator = await client.streaming()
for await (const event of iterator) {
  console.log(event.message)
}

// Or with lifecycle callbacks
import { consumeEventIterator } from '@orpc/client'

const cancel = consumeEventIterator(client.streaming(), {
  onEvent: (event) => console.log(event),
  onError: (error) => console.error(error),
  onSuccess: (value) => console.log(value),
  onFinish: (state) => console.log(state),
})
```

Stop manually with `AbortController` or `iterator.return()`.

## Key Points

- No extra configuration needed — streaming works out of the box
- Use `withEventMeta` for resume support with `lastEventId`
- `EventPublisher` is lightweight and synchronous; use `Publisher` helper for resume support
- Client Retry Plugin enables automatic reconnection like EventSource
- `return` signals successful completion; standard SSE clients will auto-reconnect

<!--
Source references:
- https://orpc.unnoq.com/docs/event-iterator
- https://orpc.unnoq.com/docs/client/event-iterator
-->
