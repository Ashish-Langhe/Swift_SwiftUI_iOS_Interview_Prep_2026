# Day 19: Strict Concurrency

## What Strict Concurrency Means

Strict concurrency is Swift's compiler checking for unsafe concurrent access.

It catches issues like:

- Accessing actor-isolated state from the wrong context
- Capturing non-sendable values in concurrent closures
- Sharing mutable state across tasks
- Calling main-actor methods from nonisolated synchronous code

## Why It Matters

Race bugs are hard to reproduce.

Swift tries to move many of those bugs from runtime to compile time.

```swift
@MainActor
final class ViewModel {
    var title = ""
}

func update(_ viewModel: ViewModel) {
    // viewModel.title = "Loaded" can be rejected outside main actor.
}
```

## Swift 6 Language Mode

Swift 6 language mode can turn many data-race safety issues into errors.

Earlier Swift versions often showed warnings through strict concurrency flags. Swift 6 makes correctness stricter when the language mode is enabled.

## Common Diagnostics

Actor-isolated call:

```swift
@MainActor
func updateUI() { }

func run() {
    // updateUI() // Error from nonisolated sync context.
}
```

Fix:

```swift
func run() {
    Task {
        await updateUI()
    }
}
```

Non-sendable capture:

```swift
final class MutableBox {
    var value = 0
}

func run(box: MutableBox) {
    Task {
        print(box.value)
    }
}
```

Fix by using actor isolation, immutable snapshots, or sendable types.

## Migration Mindset

Do not blindly silence warnings.

Ask:

- Who owns this mutable state?
- Does this cross an actor or task boundary?
- Should this be a value snapshot?
- Should this type be an actor?
- Should this code be `@MainActor`?
- Is this class truly sendable?

## Common Fix Patterns

Use `@MainActor` for UI state:

```swift
@MainActor
final class SettingsViewModel: ObservableObject { }
```

Use actors for shared mutable services:

```swift
actor CacheStore { }
```

Use value snapshots:

```swift
let snapshot = UserSnapshot(user: user)
Task {
    await logger.log(snapshot)
}
```

Use `Sendable`:

```swift
struct UserSnapshot: Sendable { }
```

## Common Mistakes

- Adding `@preconcurrency` or `@unchecked Sendable` everywhere
- Marking everything `@MainActor`
- Moving all work to detached tasks
- Ignoring protocol isolation
- Treating warnings as compiler annoyance instead of design feedback

## Modern Swift 6.x Notes

Swift 6 announced data-race safety as a major language feature. Swift 6.2 then made this easier to adopt through approachable concurrency features and migration tooling.

The goal is progressive disclosure: simple UI code should be easier, advanced concurrent code remains explicit.

## Junior Interview Answer

Strict concurrency means Swift checks more concurrency safety rules at compile time.

## Mid-Level Interview Answer

Strict concurrency catches unsafe actor access, non-sendable sharing, and potential data races. Fixes usually involve actors, `@MainActor`, `Sendable`, or immutable snapshots.

## Senior Interview Answer

Strict concurrency is a design pressure toward explicit ownership and isolation. I migrate incrementally, separate UI state from services, replace shared mutable references with actors or immutable values, and avoid unsafe escape hatches unless fully justified.

## Practice

1. Fix a main-actor call from nonisolated code.
2. Replace a non-sendable capture with a value snapshot.
3. Decide whether a shared cache should be an actor.
4. Explain why marking everything `@MainActor` is not a real migration.
