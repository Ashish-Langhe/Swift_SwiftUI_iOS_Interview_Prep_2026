# Day 8: Methods

## Core Idea

Methods are functions inside types.

```swift
struct User {
    let name: String

    func greeting() -> String {
        "Hello, \(name)"
    }
}
```

## Real iOS Use Cases

```swift
struct Transaction {
    let amount: Decimal

    func isLargeExpense() -> Bool {
        amount > 10_000
    }
}
```

## Interview Levels

Junior:

A method is a function inside a type.

Senior:

Methods should represent behavior that belongs to the type. Keep formatting and domain logic close to the model when appropriate, but avoid making models know UI details.

## Quick Notes

- Method belongs to type
- Can read properties
- Use for type behavior
- Keep responsibilities clear

## Interview Depth

Junior answer: A method is a function inside a type.

Mid-level answer: Methods represent behavior that belongs to the type. They can use the type's properties and keep related logic together.

Senior answer: Put behavior where ownership makes sense. Domain methods belong on domain models; UI formatting may belong in view models. Avoid turning models into service objects with too many responsibilities.

iOS use case:

```swift
struct FeatureFlag {
    let key: String
    let isEnabled: Bool

    func canShowFeature() -> Bool {
        isEnabled
    }
}
```

Common mistakes:

- Creating global helper functions for behavior that belongs to a type.
- Putting networking inside simple models.
- Making methods mutate without `mutating`.

Practice:

1. Add method to check ownership.
2. Move helper into struct.
3. Explain method vs function.
