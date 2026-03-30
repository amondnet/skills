---
name: integrations-ai-sdk
description: oRPC integration with Vercel AI SDK — streaming, tool creation, and chat patterns
---

# AI SDK Integration

Seamlessly integrate Vercel AI SDK (v5.0+) with oRPC for AI-powered features.

## Server — Stream to Event Iterator

```ts
import { os, streamToEventIterator, type } from '@orpc/server'
import { convertToModelMessages, streamText, UIMessage } from 'ai'
import { google } from '@ai-sdk/google'

export const chat = os
  .input(type<{ chatId: string, messages: UIMessage[] }>())
  .handler(async ({ input }) => {
    const result = streamText({
      model: google('gemini-1.5-flash'),
      system: 'You are a helpful assistant.',
      messages: await convertToModelMessages(input.messages),
    })
    return streamToEventIterator(result.toUIMessageStream())
  })
```

## Client — Event Iterator to Stream

```tsx
import { useChat } from '@ai-sdk/react'
import { eventIteratorToUnproxiedDataStream } from '@orpc/client'

const { messages, sendMessage, status } = useChat({
  transport: {
    async sendMessages(options) {
      return eventIteratorToUnproxiedDataStream(await client.chat({
        chatId: options.chatId,
        messages: options.messages,
      }, { signal: options.abortSignal }))
    },
    reconnectToStream() { throw new Error('Unsupported') },
  },
})
```

**Important:** Use `eventIteratorToUnproxiedDataStream` (not `eventIteratorToStream`) because AI SDK uses `structuredClone` which doesn't support proxied data.

## `createTool` — Procedure as AI SDK Tool

```ts
import { createTool, AI_SDK_TOOL_META_SYMBOL, AiSdkToolMeta } from '@orpc/ai-sdk'

interface ORPCMeta extends AiSdkToolMeta {}
const base = os.$meta<ORPCMeta>({})

const getWeather = base
  .meta({ [AI_SDK_TOOL_META_SYMBOL]: { title: 'Get Weather' } })
  .route({ summary: 'Get weather for a location' })
  .input(z.object({ location: z.string() }))
  .output(z.object({ location: z.string(), temperature: z.number() }))
  .handler(async ({ input }) => ({
    location: input.location,
    temperature: 72 + Math.floor(Math.random() * 21) - 10,
  }))

const tool = createTool(getWeather, { context: {} })
```

## `implementTool` — Contract as AI SDK Tool

```ts
import { implementTool } from '@orpc/ai-sdk'

const tool = implementTool(getWeatherContract, {
  execute: async ({ location }) => ({ location, temperature: 72 }),
})
```

Requires a contract with an `input` schema defined.

## Key Points

- `streamToEventIterator` converts AI SDK streams to oRPC event iterators
- `eventIteratorToUnproxiedDataStream` converts back on the client
- Use `route.summary` as AI SDK tool description
- Use `[AI_SDK_TOOL_META_SYMBOL].title` for tool title
- Both procedures and contracts can be converted to AI SDK tools

<!--
Source references:
- https://orpc.unnoq.com/docs/integrations/ai-sdk
-->
