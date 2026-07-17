# Day 11: Stored Properties

## Core Idea

Stored properties keep values inside a type.

```swift
struct User {
    let id: String
    var name: String
}
```

`let` means the property is fixed after initialization. `var` means it can change.

## Real iOS Use Cases

```swift
struct ProfileViewModel {
    let title: String
    let subtitle: String
    let imageURL: URL?
}
```

Stored properties are common in models, view models, configuration objects, and app state.

## Modern Swift 6.x Notes

Stored property mutability matters more with Swift 6 concurrency checking. Immutable stored properties are easier to reason about across tasks. Mutable stored properties inside shared reference types should be actor-isolated, main-actor-bound, or otherwise protected.

## Interview Levels

Junior: A stored property stores data in a type.

Senior: Stored properties express state and ownership. Use `let` whenever possible, and make mutability intentional because it affects safety, testing, and concurrency.

## Quick Notes

- Stored properties hold values.
- Use `let` for immutable state.
- Use `var` only when the property changes.
- Struct stored properties participate in value semantics.

## Interview Depth

Junior answer: Stored properties store values inside a type.

Mid-level answer: Stored properties define a type's state. Use `let` for values that should not change after initialization and `var` for values that genuinely change.

Senior answer: Stored properties are state ownership. Their access level and mutability should enforce invariants. In Swift 6 concurrency, mutable shared state needs isolation.

iOS use case:

```swift
struct AccountSummary {
    let id: String
    private(set) var balance: Decimal
}
```

Common mistakes: public mutable state, optional properties instead of valid initialization, mutable identity.

Practice: choose `let` vs `var`, add `private(set)`, explain state ownership.
