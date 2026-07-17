# Day 8: Stored Properties

## Core Idea

Stored properties hold values inside a type.

```swift
struct Product {
    let id: String
    var title: String
    var price: Decimal
}
```

`let` properties are immutable after initialization.

## Real iOS Use Cases

```swift
struct UserRowViewModel {
    let title: String
    let subtitle: String
    let avatarURL: URL?
}
```

## Interview Levels

Junior:

Stored properties store data in a struct or class.

Senior:

Property mutability should match domain rules. Immutable stored properties make models safer and easier to use with concurrency.

## Quick Notes

- Stored property holds data
- `let` means fixed after init
- `var` means mutable
- Good models express business rules through mutability

## Interview Depth

Junior answer: Stored properties are variables or constants stored inside a type.

Mid-level answer: Stored properties define the state of a struct or class. `let` properties are set during initialization and cannot change afterward.

Senior answer: Stored property mutability should match domain rules. If an ID should never change, make it `let`. If external code should read but not write state, use `private(set)`.

iOS use case:

```swift
struct Session {
    let userId: String
    var lastRefreshDate: Date
}
```

Common mistakes:

- Making identity mutable.
- Exposing mutable state publicly.
- Using optionals instead of valid initialization.

Practice:

1. Decide `let` vs `var` for a user model.
2. Use `private(set)`.
3. Explain stored vs computed.
