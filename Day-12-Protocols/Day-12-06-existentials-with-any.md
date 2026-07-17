# Day 12: Existentials With Any

## Core Idea

`any Protocol` means a value can hold any concrete type conforming to the protocol.

```swift
protocol AnalyticsTracking {
    func track(_ event: String)
}

let tracker: any AnalyticsTracking = ConsoleTracker()
```

## Why `any` Matters

It makes existential type erasure explicit.

## Real iOS Use Cases

- Heterogeneous arrays
- Runtime-selected services
- Dependency storage

## Modern Swift 6.x Notes

Swift 6.2 notes mention Embedded Swift support for `any` types for class-constrained protocols. Modern Swift encourages writing `any` explicitly when using existential values.

## Interview Levels

Junior: `any` stores any conforming type.

Senior: `any` has dynamic dispatch and hides the concrete type. It is flexible but may lose associated type information and optimization opportunities.

## Quick Notes

- `any Protocol` is existential.
- Hides concrete type.
- Useful for storage.
- Not the same as generics.

## Interview Depth

Junior answer: `any Protocol` can store any value that conforms to the protocol.

Mid-level answer: `any` makes existential usage explicit. It hides the concrete type behind a protocol value.

Senior answer: Existentials are flexible but trade away some static type information. Use them for storage and runtime polymorphism; use generics when you want compile-time type preservation.

iOS use case:

```swift
let tracker: any AnalyticsTracking = ConsoleAnalyticsTracker()
```

Common mistakes: confusing `any` and `some`, using `any` when generic would be clearer, losing associated type relationships.

Practice: store protocol value, compare with generic function, explain existential.
