# Day 8: Computed Properties

## Core Idea

Computed properties calculate values instead of storing them.

```swift
struct Rectangle {
    let width: Double
    let height: Double

    var area: Double {
        width * height
    }
}
```

## Getter And Setter

```swift
struct Temperature {
    var celsius: Double

    var fahrenheit: Double {
        get { celsius * 9 / 5 + 32 }
        set { celsius = (newValue - 32) * 5 / 9 }
    }
}
```

## Real iOS Use Cases

```swift
struct User {
    let firstName: String
    let lastName: String

    var fullName: String {
        "\(firstName) \(lastName)"
    }
}
```

## Interview Levels

Junior:

A computed property calculates a value.

Senior:

Computed properties are best for cheap derived values. Expensive work should be cached, moved to a method, or made explicit.

## Quick Notes

- No stored value
- Calculates from other data
- Can be get-only or get/set
- Avoid expensive hidden work

## Interview Depth

Junior answer: A computed property calculates a value when accessed.

Mid-level answer: Computed properties are useful for derived values like full name, display title, validation state, or formatted text.

Senior answer: A computed property should feel like property access, not hidden work. Expensive calculations, async operations, and throwing behavior should usually be methods.

iOS use case:

```swift
struct LoginForm {
    var email: String
    var password: String

    var canSubmit: Bool {
        email.contains("@") && password.count >= 8
    }
}
```

Common mistakes:

- Doing network calls in computed properties.
- Hiding expensive sorting/filtering.
- Adding setters when a method would be clearer.

Practice:

1. Add `fullName`.
2. Add `canSubmit`.
3. Explain property vs method.
