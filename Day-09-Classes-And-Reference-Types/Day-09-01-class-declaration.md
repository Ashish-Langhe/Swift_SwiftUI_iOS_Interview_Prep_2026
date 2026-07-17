# Day 9: Class Declaration

## Core Idea

A class defines a reference type.

```swift
final class UserSession {
    var token: String?
}
```

Instances are references to shared objects.

## Real iOS Use Cases

- View controllers
- Services
- Coordinators
- Managers
- Shared session state

## Modern Swift 6.x Notes

Shared mutable class instances require more care with Swift concurrency. Use actors, main actor isolation, locks, or immutable snapshots when needed.

## Interview Levels

Junior:

A class is a type that can have properties and methods.

Senior:

Classes provide identity and reference semantics. I use them when shared identity matters, not just as a default model type.

## Quick Notes

- Class is reference type
- Supports inheritance
- Can have deinit
- Good for identity-based objects

## Interview Depth

Junior answer: A class is a type with properties and methods.

Mid-level answer: Classes are reference types, which means variables can point to the same object. Classes support inheritance and deinitializers.

Senior answer: Use classes when identity matters, when shared mutable state is intentional, or when a framework requires subclassing. Do not use classes for every model by default.

iOS use case:

```swift
@MainActor
final class ProfileViewModel {
    private(set) var state: ProfileState = .loading
}
```

Common mistakes:

- Not marking app classes `final`.
- Using classes for plain values.
- Sharing mutable class state across tasks.

Practice:

1. Declare a final service class.
2. Explain class vs struct.
3. Explain when identity matters.
