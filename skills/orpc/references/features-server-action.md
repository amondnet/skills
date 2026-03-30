---
name: features-server-action
description: React Server Actions integration with useServerAction, optimistic updates, and form handling
---

# Server Action

Integrate oRPC procedures with React Server Actions using `.actionable`.

## Server Side

```ts
'use server'

import { redirect } from 'next/navigation'

export const ping = os
  .input(z.object({ name: z.string() }))
  .handler(async ({ input }) => `Hello, ${input.name}`)
  .actionable({
    context: async () => ({}),
    interceptors: [
      onSuccess(async output => redirect('/some-where')),
      onError(async error => console.error(error)),
    ],
  })
```

**Tip:** Use execution context instead of initial context with Server Actions.

## Client Side

```tsx
'use client'
import { ping } from './actions'

const [error, data] = await ping({ name: 'John' })

// Type-safe error handling
if (error) {
  if (error.defined) {
    console.log(error.data) // Typed error data
  }
}
```

## `@orpc/react` Package

### `useServerAction` Hook

```tsx
import { useServerAction } from '@orpc/react/hooks'
import { isDefinedError, onError } from '@orpc/client'

const { execute, data, error, status } = useServerAction(someAction, {
  interceptors: [
    onError((error) => {
      if (isDefinedError(error)) console.error(error.data)
    }),
  ],
})
```

### `useOptimisticServerAction` Hook

```tsx
import { useOptimisticServerAction } from '@orpc/react/hooks'

const { execute, optimisticState } = useOptimisticServerAction(someAction, {
  optimisticPassthrough: todos,
  optimisticReducer: (currentState, newTodo) => [...currentState, newTodo],
  interceptors: [
    onSuccessDeferred(({ data }) => setTodos(prev => [...prev, data])),
  ],
})
```

### `createFormAction` Utility

Handles form submissions with bracket notation deserialization:

```tsx
import { createFormAction } from '@orpc/react'

const formAction = createFormAction(procedure, {
  interceptors: [onSuccess(async () => redirect('/done'))],
})

// In JSX: <form action={formAction}>
//   <input name="user[name]" />
//   <input name="user[age]" type="number" />
// </form>
```

Auto-converts ORPCError 401/403/404 to Next.js `unauthorized`/`forbidden`/`notFound`.

### Form Data Utilities

```tsx
import { getIssueMessage, parseFormData } from '@orpc/react'

// Parse form data with bracket notation
execute(parseFormData(formData))

// Show validation errors per field
<span>{getIssueMessage(error, 'user[name]')}</span>
```

## Key Points

- `.actionable()` makes any procedure a Server Action
- Special errors (`redirect`, `notFound`) only supported in Next.js and TanStack Start
- `onSuccessDeferred` defers execution for state updates in optimistic patterns
- `createFormAction` uses bracket notation for nested form data

<!--
Source references:
- https://orpc.unnoq.com/docs/server-action
-->
