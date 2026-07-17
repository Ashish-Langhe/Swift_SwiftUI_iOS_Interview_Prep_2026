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

## Deep Dive Expansion

### Why This Topic Matters In Real Apps

Strict Concurrency is not only a syntax topic. In production Swift, it affects actor isolation, Sendable transfer, strict checking, migration, and data-race safety. A junior developer should be able to recognize the syntax and explain the basic behavior. A mid-level developer should know when the feature improves clarity or safety. A senior developer should understand how it changes API design, testability, performance, or long-term maintenance.

In iOS work, this usually appears while token stores, caches, UI models, Swift 6 migration, and shared services. The important question is not "Can I write the syntax?" The important question is "Does this code make invalid states harder, make ownership clearer, and make future changes safer?"

### Beginner To Senior Progression

Beginner level:

- Define the concept in plain language.
- Write the smallest working example.
- Recognize the compiler error when the feature is used incorrectly.
- Explain what happens at the call site.

Mid-level level:

- Choose this feature over a nearby alternative for a clear reason.
- Handle edge cases, nil/error/cancellation/performance concerns where relevant.
- Keep the code readable for the next developer.
- Write tests around the behavior, not just the implementation detail.

Senior level:

- Explain the design tradeoff and the failure mode it prevents.
- Understand how this feature behaves across module, actor, memory, or API boundaries.
- Design examples that scale from a small screen to a larger feature.
- Avoid exposing implementation details as permanent API.

### Production-Style Example Pattern

Use this checklist when applying Strict Concurrency in an app feature:

1. Identify the owner of the data or behavior.
2. Decide whether the value should be mutable, immutable, optional, throwing, async, isolated, or private.
3. Keep the public surface small and intention-revealing.
4. Add one realistic failure path, not only the happy path.
5. Check whether the code is still understandable from the call site.

```swift
struct FeatureState: Equatable {
    var isLoading: Bool
    var message: String?
    var canRetry: Bool
}

func makeInitialState() -> FeatureState {
    FeatureState(isLoading: false, message: nil, canRetry: false)
}
```

This small pattern is intentionally simple: define the state, control mutation through a narrow function, and make the result easy to inspect in tests.

### Edge Cases To Think About

- What happens when the input is empty, nil, duplicated, delayed, or invalid?
- What happens when this code is called repeatedly from a scrolling list or fast-changing UI?
- Does this API expose too much mutable state?
- Does the implementation assume a specific ordering, lifetime, actor, or thread?
- Will this still be easy to test after the feature grows?

### Topic-Specific Senior Notes

- Always separate task lifetime from object lifetime; a task can outlive the screen that started it.
- State crossing a concurrency boundary should be immutable, actor-isolated, or clearly Sendable.

### Common Interview Follow-Ups

Be ready for these follow-ups:

- Why did you choose this approach instead of the simpler alternative?
- What bug could happen if this is implemented carelessly?
- How would this behave in a large codebase with multiple modules?
- How would you test this without relying on UI screenshots?
- What changes when this code becomes async, public, generic, or shared?

### Strong Interview Framing

A strong answer should sound like this:

```text
I understand the basic syntax, but I also think about ownership and boundaries. I choose the approach that keeps state valid, makes the call site clear, and avoids unnecessary coupling. In a production iOS app, I would also consider testing, cancellation or error behavior where relevant, and whether this should remain an implementation detail or become part of a public API.
```

### Extra Practice

1. Write a minimal example that shows Strict Concurrency correctly.
2. Write a second example that demonstrates a common mistake.
3. Refactor the mistake into a safer version.
4. Explain the difference between the beginner solution and the production-ready solution.
5. Add one test case or reasoning check that proves the behavior.

## Applied Day-Level Example

The following example is not the only way to use Strict Concurrency, but it shows the kind of production shape you should connect this topic to:

```swift
actor UserCache {
    private var storage: [User.ID: User] = [:]

    func user(id: User.ID) -> User? {
        storage[id]
    }

    func insert(_ user: User) {
        storage[user.id] = user
    }
}
```

Modern Swift concurrency asks who owns mutable state. If many tasks need the same cache, actor isolation is clearer than hoping callers use it safely.

### How To Study This Example

- Identify which part is syntax and which part is design.
- Ask what invalid state the example prevents.
- Ask what would need to change if the feature became async, generic, public, or shared across modules.
- Rewrite the example with one deliberate bug, then explain how a reviewer should catch it.
- Turn the example into a two-minute interview explanation with tradeoffs.

