---
name: advanced-custom-plugins
description: Building custom oRPC handler and link plugins with interceptors and lifecycle hooks
---

# Building Custom Plugins

Plugins are collections of interceptors that extend handler or link functionality.

## Plugin Structure

```ts
export class MyPlugin<T extends MyPluginContext> implements StandardHandlerPlugin<T> {
  order = 10  // controls loading order (optional)

  init(options: StandardHandlerOptions<T>): void {
    options.rootInterceptors ??= []
    options.clientInterceptors ??= []

    options.rootInterceptors.push(async (interceptorOptions) => {
      // Before handler
      const result = await interceptorOptions.next({ ...interceptorOptions })
      // After handler
      return result
    })
  }
}
```

## Handler Plugins (Server)

Extend RPCHandler, OpenAPIHandler, or custom handlers. Hook into the handler lifecycle:

- `rootInterceptors` — Run after request conversion, before routing
- `interceptors` — Run after routing, before input decoding
- `clientInterceptors` — Run during procedure execution

See [built-in handler plugins](https://github.com/middleapi/orpc/tree/main/packages/server/src/plugins) for examples.

## Link Plugins (Client)

Extend RPCLink, OpenAPILink, or custom links. Hook into the link lifecycle:

- `interceptors` — Run around request encoding/decoding
- `clientInterceptors` — Run around request sending
- `adapterInterceptors` — Run around the actual fetch call

See [built-in link plugins](https://github.com/middleapi/orpc/tree/main/packages/client/src/plugins) for examples.

## Communication Between Interceptors

Use unique symbols to share data between interceptors:

```ts
const MY_SYMBOL = Symbol('my-data')

// In rootInterceptors: collect data
options.rootInterceptors.push(async (opts) => {
  return opts.next({ ...opts, context: { ...opts.context, [MY_SYMBOL]: someData } })
})

// In clientInterceptors: use data
options.clientInterceptors.push(async (opts) => {
  const data = opts.context[MY_SYMBOL]
  // ...
})
```

## Plugin Order

The `order` property controls loading order, not interceptor execution order.

- Higher order → loads first
- Use `.unshift` for earlier interceptor execution, `.push` for later
- Keep `order < 1_000_000` to avoid conflicts with built-in plugins
- Only define `order` when your interceptors must always run before/after others

## Key Points

- A plugin's `init` method receives the handler/link options to modify
- Handler lifecycle: request → rootInterceptors → interceptors → clientInterceptors → response
- Link lifecycle: input → interceptors → clientInterceptors → adapterInterceptors → output
- Use context injection patterns to communicate between interceptor layers

<!--
Source references:
- https://orpc.unnoq.com/docs/advanced/building-custom-plugins
-->
