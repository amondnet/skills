---
name: core-consumers
description: Consumer widgets that bridge Flutter widget tree and Riverpod provider tree — Consumer, ConsumerWidget, ConsumerStatefulWidget, HookConsumerWidget
---

# Consumers

Consumers are widgets that bridge the Widget tree and the Provider tree by providing access to a Ref.

## Three Flavors

### Consumer (builder widget)

Works with any widget, no subclassing needed:

```dart
class MyWidget extends StatelessWidget {
  @override
  Widget build(BuildContext context) {
    return Consumer(
      builder: (context, ref, _) {
        final value = ref.watch(myProvider);
        return Text(value.toString());
      },
    );
  }
}
```

### ConsumerWidget (replaces StatelessWidget)

```dart
class MyWidget extends ConsumerWidget {
  @override
  Widget build(BuildContext context, WidgetRef ref) {
    final value = ref.watch(myProvider);
    return Text(value.toString());
  }
}
```

### ConsumerStatefulWidget (replaces StatefulWidget)

```dart
class MyWidget extends ConsumerStatefulWidget {
  @override
  ConsumerState<MyWidget> createState() => _MyWidgetState();
}

class _MyWidgetState extends ConsumerState<MyWidget> {
  @override
  Widget build(BuildContext context) {
    final value = ref.watch(myProvider); // this.ref available
    return Text(value.toString());
  }
}
```

## With flutter_hooks

Use `hooks_riverpod` package for combined hooks + Riverpod:

- `HookConsumerWidget` — combines `HookWidget` + `ConsumerWidget`
- `HookConsumer` — builder combining both

```dart
class MyWidget extends HookConsumerWidget {
  @override
  Widget build(BuildContext context, WidgetRef ref) {
    final controller = useAnimationController(duration: Duration(seconds: 2));
    final value = ref.watch(myProvider);
    return Text(value.toString());
  }
}
```

## Which to Use?

- **Consumer**: Most flexible, slightly more verbose. Good if you don't want Riverpod to hijack base classes.
- **ConsumerWidget**: Recommended default for stateless widgets.
- **ConsumerStatefulWidget**: When you need `State` lifecycle.

## Why Not context.watch?

Riverpod requires dedicated Consumer types (not `context.watch` like `package:provider`) because `BuildContext` cannot reliably track listener removal, which would break auto-dispose and cause memory leaks/background work.

<!--
Source references:
- https://riverpod.dev/docs/concepts2/consumers
-->
