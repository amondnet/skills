---
name: features-mutations
description: Experimental Mutations API for tracking side-effect state (loading/error/success) in UI — defining, listening, triggering, scoping, and resetting mutations
---

# Mutations (Experimental)

Mutations enable UIs to react to side-effects (form submission, button clicks) by tracking loading/error/success states without polluting provider state.

> API may change in breaking ways without a major version bump.

## Defining a Mutation

```dart
// Global or static final on a Notifier
final addTodo = Mutation<Todo>();
```

The generic type enables UI to access the result value.

## Listening to a Mutation

Watch mutations in Consumer/providers using `ref.watch`:

```dart
class AddTodoButton extends ConsumerWidget {
  @override
  Widget build(BuildContext context, WidgetRef ref) {
    final addTodoState = ref.watch(addTodo);

    return switch (addTodoState) {
      MutationIdle() => ElevatedButton(
        onPressed: () { /* trigger */ },
        child: Text('Submit'),
      ),
      MutationPending() => CircularProgressIndicator(),
      MutationError(:final error) => Text('Failed: $error'),
      MutationSuccess(:final value) => Text('Added: ${value.title}'),
    };
  }
}
```

## Triggering a Mutation

Use `Mutation.run` with an async callback:

```dart
onPressed: () {
  addTodo.run(ref, (tsx) async {
    await tsx.get(todoListProvider.notifier).addTodo('New Todo');
    return newTodo; // Must match Mutation<Todo> type
  });
}
```

Use `tsx.get` (not `ref.read`) — it keeps providers alive until the mutation completes.

## Mutation States

| State             | Meaning                         |
|-------------------|---------------------------------|
| `MutationIdle`    | Not started or reset            |
| `MutationPending` | In progress (loading)           |
| `MutationSuccess` | Completed successfully          |
| `MutationError`   | Failed with error               |

## Scoped Mutations (Keyed)

For multiple instances (e.g., delete button per item):

```dart
final deleteTodo = Mutation<void>();

// Key by item ID
final deleteState = ref.watch(deleteTodo(todoId));
```

## Resetting

Mutations auto-reset to `MutationIdle` when:
- Completed (success or error) and all listeners removed
- Similar to auto-dispose behavior

Manual reset:

```dart
deleteTodo.reset();
// or keyed:
deleteTodo(todoId).reset();
```

<!--
Source references:
- https://riverpod.dev/docs/concepts2/mutations
-->
