# Day 19: `Sendable`

## What `Sendable` Means

`Sendable` means values of a type can be safely passed across concurrency boundaries.

```swift
struct User: Sendable {
    let id: String
    let name: String
}
```

Immutable value types are usually naturally sendable.

## Why `Sendable` Matters

When data crosses from one task or actor to another, Swift needs to know that sharing it will not create data races.

```swift
Task {
    await service.save(user)
}
```

If `user` is shared into concurrent work, its type may need to be `Sendable`.

## Value Types

Good:

```swift
struct Product: Sendable {
    let id: UUID
    let title: String
    let price: Decimal
}
```

This is safe because properties are value-like and immutable.

## Classes And Sendable

Classes are reference types, so `Sendable` is harder.

```swift
final class Configuration: Sendable {
    let baseURL: URL

    init(baseURL: URL) {
        self.baseURL = baseURL
    }
}
```

An immutable final class can be sendable.

Mutable classes are usually not safely sendable unless protected by synchronization or actor isolation.

## `@unchecked Sendable`

```swift
final class LockedStore: @unchecked Sendable {
    private let lock = NSLock()
    private var values: [String: String] = [:]
}
```

`@unchecked Sendable` tells the compiler:

```text
Trust me. I manually guarantee thread safety.
```

Use it rarely and document why it is safe.

## `@Sendable` Closures

`@Sendable` closures are closures that can safely cross concurrency boundaries.

```swift
func perform(_ operation: @Sendable @escaping () async -> Void) {
    Task {
        await operation()
    }
}
```

Capturing non-sendable mutable state inside a `@Sendable` closure can produce warnings or errors under strict concurrency.

## Real iOS Example

```swift
struct AnalyticsEvent: Sendable {
    let name: String
    let properties: [String: String]
}

actor AnalyticsStore {
    private var events: [AnalyticsEvent] = []

    func append(_ event: AnalyticsEvent) {
        events.append(event)
    }
}
```

Events cross into actor isolation safely.

## Common Mistakes

- Adding `@unchecked Sendable` to silence compiler errors
- Making mutable classes sendable without protection
- Capturing `self` in `@Sendable` closures without considering isolation
- Ignoring sendability in public async APIs
- Assuming all structs are automatically safe if they contain reference properties

## Modern Swift 6.x Notes

Swift 6 language mode strengthens data-race safety diagnostics. `Sendable` becomes central when values cross tasks, actors, and concurrent closures.

Swift 6.2 also includes diagnostics around sendable metatypes and improved migration tooling for upcoming concurrency behavior.

## Junior Interview Answer

`Sendable` means a type's values can be safely shared across concurrency boundaries.

## Mid-Level Interview Answer

I make immutable DTOs and domain models `Sendable` when they cross tasks or actors. I avoid making mutable classes sendable unless they are actor-isolated or internally synchronized.

## Senior Interview Answer

`Sendable` is a compile-time contract for data transfer between concurrency domains. I prefer immutable value types, avoid reference sharing, treat `@unchecked Sendable` as a last resort, and include sendability in public async API design.

## Practice

1. Make an immutable domain model `Sendable`.
2. Explain why a mutable class is not automatically sendable.
3. Write a safe `@Sendable` closure.
4. Replace an unsafe reference type with a value snapshot.

## Deep Dive Expansion

### Why This Topic Matters In Real Apps

Sendable is not only a syntax topic. In production Swift, it affects actor isolation, Sendable transfer, strict checking, migration, and data-race safety. A junior developer should be able to recognize the syntax and explain the basic behavior. A mid-level developer should know when the feature improves clarity or safety. A senior developer should understand how it changes API design, testability, performance, or long-term maintenance.

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

Use this checklist when applying Sendable in an app feature:

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

1. Write a minimal example that shows Sendable correctly.
2. Write a second example that demonstrates a common mistake.
3. Refactor the mistake into a safer version.
4. Explain the difference between the beginner solution and the production-ready solution.
5. Add one test case or reasoning check that proves the behavior.

## Applied Day-Level Example

The following example is not the only way to use Sendable, but it shows the kind of production shape you should connect this topic to:

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

