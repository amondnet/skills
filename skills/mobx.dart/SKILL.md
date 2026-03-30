---
name: mobx-dart
description: MobX.dart reactive state management for Dart/Flutter — observables, actions, reactions, Observer widget, store patterns, and code generation with mobx_codegen
metadata:
  author: Minsu Lee
  version: "2026.3.30"
  source: Generated from https://github.com/amondnet/mobx.dart, scripts located at https://github.com/amondnet/skills
---

> The skill is based on mobx v2.6.0 / flutter_mobx v2.0.4, generated at 2026-03-30.

MobX.dart is a reactive state management library for Dart and Flutter built around three core concepts: Observables (reactive state), Actions (state mutations), and Reactions (side-effects). It uses `mobx_codegen` for annotation-based code generation to minimize boilerplate.

## Core References

| Topic | Description | Reference |
|-------|-------------|-----------|
| Store & Code Generation | Store class pattern, annotations, build_runner setup | [core-store-and-codegen](references/core-store-and-codegen.md) |
| Observables | Observable, Computed, @observable, @readonly, @computed, reactive extensions | [core-observables](references/core-observables.md) |
| Actions | @action, runInAction, untracked, transaction, async actions | [core-actions](references/core-actions.md) |
| Reactions | autorun, reaction, when, asyncWhen, custom schedulers | [core-reactions](references/core-reactions.md) |

## Features

### Flutter Integration

| Topic | Description | Reference |
|-------|-------------|-----------|
| Observer Widget | Observer, Observer.withBuiltChild, ReactionBuilder for reactive UI | [features-observer-widget](references/features-observer-widget.md) |
| Reactive Collections | ObservableList, ObservableMap, ObservableSet, ObservableFuture, ObservableStream, Atom | [features-reactive-collections](references/features-reactive-collections.md) |

## Best Practices

| Topic | Description | Reference |
|-------|-------------|-----------|
| Store Organization | Widget-Store-Service triad, store hierarchy, inter-store communication, Provider integration | [best-practices-store-organization](references/best-practices-store-organization.md) |
| Reactivity Rules | When MobX reacts, tracking pitfalls, Observable vs plain collections, form validation pattern | [best-practices-reactivity-rules](references/best-practices-reactivity-rules.md) |
| JSON Serialization | json_serializable integration with custom converters for observable collections | [best-practices-json-serialization](references/best-practices-json-serialization.md) |

## Advanced

| Topic | Description | Reference |
|-------|-------------|-----------|
| Context & Config | ReactiveContext, ReactiveConfig, read/write policies, error boundaries | [advanced-context-and-config](references/advanced-context-and-config.md) |
| Spy (Debugging) | Spy API for tracing reactive events and debugging | [advanced-spy](references/advanced-spy.md) |
