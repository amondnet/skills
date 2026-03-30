---
name: features-openapi
description: oRPC OpenAPI specification generation, OpenAPIHandler, routing, and input/output structure
---

# OpenAPI

oRPC fully supports OpenAPI 3.1.1 for spec generation, RESTful handling, and client communication.

## OpenAPI Specification Generation

```ts
import { OpenAPIGenerator } from '@orpc/openapi'
import { ZodToJsonSchemaConverter } from '@orpc/zod'  // or @orpc/zod/zod4

const generator = new OpenAPIGenerator({
  schemaConverters: [new ZodToJsonSchemaConverter()],
})

const spec = await generator.generate(router, {
  info: { title: 'My App', version: '0.0.0' },
  servers: [{ url: 'https://api.example.com/v1' }],
})
```

Supports Zod (v3/v4), Valibot, and ArkType converters.

## OpenAPIHandler

Serves RESTful endpoints following the OpenAPI spec:

```ts
import { OpenAPIHandler } from '@orpc/openapi/fetch'
import { CORSPlugin } from '@orpc/server/plugins'

const handler = new OpenAPIHandler(router, {
  plugins: [new CORSPlugin()],
})

const { matched, response } = await handler.handle(request, {
  prefix: '/api',
  context: {}
})
```

## Routing

```ts
// Basic routing
os.route({ method: 'GET', path: '/planets/{id}', successStatus: 200 })

// Path parameters (merge with query/body into input)
os.route({ path: '/example/{id}' })
  .input(z.object({ id: z.string() }))

// Wildcard paths
os.route({ path: '/files/{+path}' })  // matches slashes

// Prefix for routers
const router = os.prefix('/planets').router({ list, find, create })

// Tags
const router = os.tag('planets').router({ /* ... */ })
```

## Input/Output Structure

### Compact Mode (default)

Combines path params with query/body into a single input object.

### Detailed Mode

Separate params, query, headers, and body:

```ts
os.route({ path: '/ping/{name}', method: 'POST', inputStructure: 'detailed' })
  .input(z.object({
    params: z.object({ name: z.string() }),
    query: z.object({ search: z.string() }),
    body: z.object({ description: z.string() }).optional(),
    headers: z.object({ 'x-custom-header': z.string() }),
  }))

// Output detailed mode
os.route({ outputStructure: 'detailed' })
  .handler(async () => ({
    status: 201,
    headers: { 'x-custom-header': 'value' },
    body: { message: 'Created' },
  }))
```

## Operation Metadata

```ts
const ping = os
  .route({
    operationId: 'ping',
    summary: 'Ping endpoint',
    description: 'Health check',
    deprecated: false,
    tags: ['health'],
    successDescription: 'Pong',
  })
  .handler(() => 'pong')
```

Extend with `oo.spec` for security and other OpenAPI operation properties:

```ts
import { oo } from '@orpc/openapi'

const base = os.errors({
  UNAUTHORIZED: oo.spec({ data: z.any() }, { security: [{ 'api-key': [] }] })
})
```

## Common Schemas

Define reusable `$ref` components:

```ts
const spec = await generator.generate(router, {
  commonSchemas: {
    User: { schema: UserSchema },
    InputPet: { strategy: 'input', schema: PetSchema },
    UndefinedError: { error: 'UndefinedError' },
  },
})
```

## Key Points

- RPCHandler for oRPC protocol, OpenAPIHandler for REST — can run both simultaneously
- Use `.route()` for HTTP method, path, status customization
- `inputStructure: 'detailed'` separates params/query/headers/body
- `oo.spec` on errors/middleware auto-applies OpenAPI properties to all using procedures
- Supports Zod v3 (`@orpc/zod`) and v4 (`@orpc/zod/zod4`)

<!--
Source references:
- https://orpc.unnoq.com/docs/openapi/openapi-specification
- https://orpc.unnoq.com/docs/openapi/openapi-handler
- https://orpc.unnoq.com/docs/openapi/routing
- https://orpc.unnoq.com/docs/openapi/input-output-structure
- https://orpc.unnoq.com/docs/openapi/error-handling
-->
