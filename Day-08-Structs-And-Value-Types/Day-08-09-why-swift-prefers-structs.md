# Day 8: Why Swift Prefers Structs

## Core Idea

Swift often prefers structs because they are predictable value types.

Use structs for:

- Models
- View models
- Configuration
- State
- Small domain values

Use classes when you need:

- Identity
- Shared mutable state
- Inheritance
- Reference semantics
- Objective-C interoperability

## SwiftUI Connection

SwiftUI views are structs.

```swift
struct ProfileView: View {
    var body: some View {
        Text("Profile")
    }
}
```

## Modern Swift 6.x Notes

Swift's concurrency model makes value types and immutable snapshots especially useful. Structs can conform to `Sendable` more naturally when their stored properties are safe.

## Interview Levels

Junior:

Structs are value types and are commonly used in Swift.

Senior:

Swift prefers structs because value semantics reduce hidden sharing, improve predictability, and fit SwiftUI's state-driven model. Classes are still correct when identity and shared reference behavior are required.

## Quick Notes

- Prefer structs by default
- Use classes for identity
- Great for SwiftUI and models
- Value semantics aid safety

## Interview Depth

Junior answer: Swift often uses structs because they are simple value types.

Mid-level answer: Structs are good for models, state, and SwiftUI views because they make copying and mutation predictable.

Senior answer: Swift prefers structs because they reduce shared mutable state. Classes are still correct for identity, inheritance, reference sharing, and framework requirements. The choice should be based on semantics, not habit.

iOS use case:

```swift
struct SettingsState {
    var selectedTheme: Theme
    var notificationsEnabled: Bool
}
```

Common mistakes:

- Saying structs are always better.
- Using class just to avoid `mutating`.
- Forgetting classes are needed for UIKit controllers.

Practice:

1. Choose struct/class for API model.
2. Choose struct/class for view controller.
3. Explain identity vs value.
