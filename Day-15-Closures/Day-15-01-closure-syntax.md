# Day 15: Closure Syntax

## Core Idea

A closure is a self-contained block of code.

```swift
let greet = {
    print("Hello")
}

greet()
```

Closure with parameters:

```swift
let add: (Int, Int) -> Int = { a, b in
    a + b
}
```

## Real iOS Use Cases

- Button actions
- Completion handlers
- SwiftUI view builders
- Collection transformations

## Interview Levels

Junior: A closure is like an unnamed function.

Senior: Closures are first-class function values that can capture context and be passed around as behavior.

## Quick Notes

- `{ }` syntax.
- `in` separates parameters from body.
- Can be stored and passed.

## Interview Depth

Junior answer: A closure is a block of code that can be stored or passed around.

Mid-level answer: Closures are commonly used for callbacks, button actions, animations, and collection transformations.

Senior answer: Closure syntax should balance concision and clarity. Use shorthand like `$0` for simple transformations, but named parameters for business logic.

iOS use case:

```swift
let onSave: () -> Void = {
    print("Save tapped")
}
```

Common mistakes: unreadable shorthand, complex logic inline, unclear return type.

Practice: write closure with parameters, store closure, pass closure to map.
