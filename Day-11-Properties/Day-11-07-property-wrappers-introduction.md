# Day 11: Property Wrappers Introduction

## Core Idea

Property wrappers add reusable behavior around property access.

```swift
@propertyWrapper
struct Clamped {
    private var value: Int
    let range: ClosedRange<Int>

    var wrappedValue: Int {
        get { value }
        set { value = min(max(newValue, range.lowerBound), range.upperBound) }
    }

    init(wrappedValue: Int, _ range: ClosedRange<Int>) {
        self.range = range
        self.value = min(max(wrappedValue, range.lowerBound), range.upperBound)
    }
}
```

## SwiftUI Examples

```swift
@State private var count = 0
@Binding var isPresented: Bool
```

## Real iOS Use Cases

- SwiftUI state
- UserDefaults wrappers
- Validation
- Dependency injection

## Interview Levels

Junior: A property wrapper adds behavior to a property using `@`.

Senior: Property wrappers are syntactic abstraction for repeated property access patterns. They are powerful, but overuse can hide behavior and make debugging harder.

## Quick Notes

- Uses `@propertyWrapper`.
- Exposes `wrappedValue`.
- Common in SwiftUI.
- Avoid hiding surprising side effects.

## Interview Depth

Junior answer: A property wrapper adds reusable behavior to a property using `@`.

Mid-level answer: SwiftUI uses property wrappers like `@State`, `@Binding`, and `@Environment` to manage view state and dependencies.

Senior answer: Property wrappers are abstraction over property access. They can improve consistency, but overuse can hide storage, threading, persistence, or side effects.

iOS use case:

```swift
@State private var isPresented = false
@AppStorage("hasSeenOnboarding") private var hasSeenOnboarding = false
```

Common mistakes: using wrapper without understanding ownership, writing wrappers with side effects, confusing `@State` and `@Binding`.

Practice: explain `wrappedValue`, compare State/Binding, create simple clamped wrapper.
