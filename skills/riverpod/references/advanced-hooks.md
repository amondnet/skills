---
name: advanced-hooks
description: flutter_hooks integration with Riverpod — when to use hooks, HookConsumerWidget, HookConsumer, rules of hooks, managing local widget state
---

# flutter_hooks Integration

Hooks are an optional companion to Riverpod for managing **local widget state** (animations, controllers, form state). Hooks are for ephemeral state; providers are for shared business state.

## When to Use Hooks

- Animations (`useAnimationController`)
- Text editing (`useTextEditingController`)
- Focus management (`useFocusNode`)
- Replacing `StatefulWidget` boilerplate

**Don't use hooks if you're new to Riverpod** — learn providers first.

## Setup

Install both `hooks_riverpod` and `flutter_hooks`:

```yaml
dependencies:
  hooks_riverpod: ^3.0.0
  flutter_hooks: ^0.20.0
```

## HookConsumerWidget

Combines `HookWidget` + `ConsumerWidget`:

```dart
class MyWidget extends HookConsumerWidget {
  @override
  Widget build(BuildContext context, WidgetRef ref) {
    final controller = useAnimationController(duration: Duration(seconds: 2));
    final data = ref.watch(myProvider);
    return Text(data.toString());
  }
}
```

## HookConsumer (Builder)

For use inside any widget without subclassing:

```dart
HookConsumer(
  builder: (context, ref, child) {
    final controller = useTextEditingController();
    final data = ref.watch(myProvider);
    return TextField(controller: controller);
  },
)
```

## Rules of Hooks

1. Only use inside `build` of a `HookWidget`/`HookConsumerWidget`
2. Never use conditionally (`if`) or in loops (`for`)
3. Can call multiple hooks — order must be consistent across rebuilds

## Custom Hooks

Extract reusable stateful logic:

```dart
double useFadeIn() {
  final controller = useAnimationController(duration: Duration(seconds: 2));
  useEffect(() {
    controller.forward();
    return null;
  }, const []);
  useAnimation(controller);
  return controller.value;
}
```

<!--
Source references:
- https://riverpod.dev/docs/concepts/about_hooks
-->
