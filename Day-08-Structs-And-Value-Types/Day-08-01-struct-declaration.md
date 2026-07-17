# Day 8: Struct Declaration

## Core Idea

A struct defines a value type.

```swift
struct User {
    let id: String
    var name: String
}
```

Create an instance:

```swift
let user = User(id: "u1", name: "Aarav")
```

## Real iOS Use Cases

- API models
- View models
- State snapshots
- Configuration
- Value objects

## Modern Swift 6.x Notes

Swift 6 concurrency makes immutable structs especially attractive because value snapshots are easier to reason about across tasks. Add `Sendable` when crossing concurrency domains.

## Interview Levels

Junior:

A struct groups properties and behavior.

Senior:

Structs provide value semantics, making state predictable. They are preferred for models unless identity or shared mutation is required.

## Quick Notes

- Struct is value type
- Good for models
- Can have properties and methods
- Prefer immutable `let` properties where possible

## Interview Depth

Junior answer: A struct is a type that groups related data and behavior.

Mid-level answer: Structs are value types, so assigning or passing them creates a logically separate value. They are commonly used for models, view models, and SwiftUI views.

Senior answer: Structs are the default choice for data modeling in Swift because value semantics reduce hidden sharing. They make state transitions easier to reason about, especially in SwiftUI and concurrent code.

iOS use case:

```swift
struct ExpenseRowViewModel: Identifiable {
    let id: String
    let title: String
    let amountText: String
}
```

Common mistakes:

- Using classes for simple data.
- Making every property mutable.
- Putting unrelated responsibilities in one struct.

Practice:

1. Create a struct for API response.
2. Create a struct for SwiftUI row data.
3. Explain why struct is preferred for models.
