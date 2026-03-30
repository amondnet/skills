---
name: best-practices-do-dont
description: Official Riverpod DO/DON'T guidelines — avoid widget-driven init, ephemeral state, side-effects in build, dynamic providers; prefer static provider references
---

# DO/DON'T

Official best practices enforced by `riverpod_lint`.

## AVOID Initializing Providers in Widgets

Providers should initialize themselves. Don't call init methods from `initState`.

```dart
// BAD
void initState() {
  super.initState();
  ref.read(provider).init(); // Race conditions possible
}

// GOOD — trigger in user interaction
ElevatedButton(
  onPressed: () {
    ref.read(provider).init();
    Navigator.of(context).push(...);
  },
)
```

## AVOID Providers for Ephemeral State

Don't use providers for:
- Currently selected items (scoped to a route)
- Form state (should reset on re-entry)
- Animations
- Anything Flutter manages with controllers (`TextEditingController`, etc.)

Use `flutter_hooks` or `StatefulWidget` for local widget state instead. Route-scoped state in providers breaks back button behavior.

## DON'T Perform Side-Effects During Provider Initialization

Providers represent "read" operations. Use Mutations for "write" operations.

```dart
// BAD — provider for write operation
final submitProvider = FutureProvider((ref) async {
  final formState = ref.watch(formState);
  return http.post('https://api.com', body: formState.toJson());
});

// GOOD — use Mutations or Notifier methods
```

## PREFER Static Provider References

Use providers known at compile time for better lint support:

```dart
// GOOD
ref.watch(userProvider);

// BAD — lints can't analyze dynamic providers
class Example extends ConsumerWidget {
  Example({required this.provider});
  final Provider<int> provider;

  Widget build(context, ref) {
    ref.watch(provider); // Static analysis can't help here
  }
}
```

## AVOID Dynamically Creating Providers

Providers must be top-level final variables:

```dart
// GOOD
final provider = Provider<String>((ref) => 'Hello');

// BAD — memory leaks, unexpected behavior
class Example {
  final provider = Provider<String>((ref) => 'Hello');
}
```

Static final variables on classes are allowed but not supported by codegen.

<!--
Source references:
- https://riverpod.dev/docs/root/do_dont
-->
