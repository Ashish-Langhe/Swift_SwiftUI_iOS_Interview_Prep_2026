# Day 11: Computed Properties

## Core Idea

Computed properties calculate values instead of storing them.

```swift
struct User {
    let firstName: String
    let lastName: String

    var fullName: String {
        "\(firstName) \(lastName)"
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

- Display labels
- Derived validation state
- UI enable/disable conditions

```swift
struct LoginForm {
    var email: String
    var password: String

    var canSubmit: Bool {
        email.contains("@") && password.count >= 8
    }
}
```

## Interview Levels

Junior: A computed property returns a calculated value.

Senior: Computed properties are good for cheap derived values. Expensive work, async work, or side effects should usually be methods so callers understand cost and behavior.

## Quick Notes

- No stored backing value by default.
- Can be read-only or get/set.
- Great for derived UI state.
- Avoid hidden expensive computation.

## Interview Depth

Junior answer: A computed property calculates a value when accessed.

Mid-level answer: Computed properties are great for derived values like `fullName`, `canSubmit`, or formatted state.

Senior answer: Use computed properties for cheap, synchronous, deterministic values. Expensive, throwing, or async work should usually be a method so cost is visible.

iOS use case:

```swift
var isSubmitEnabled: Bool {
    email.contains("@") && password.count >= 8
}
```

Common mistakes: doing network calls, sorting large arrays every access, hiding side effects.

Practice: create `fullName`, create `canSubmit`, decide property vs method.
