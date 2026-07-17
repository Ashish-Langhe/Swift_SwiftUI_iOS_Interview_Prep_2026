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
