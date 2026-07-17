# Day 9: Final

## Core Idea

`final` prevents subclassing or overriding.

```swift
final class APIClient { }
```

```swift
class Base {
    final func id() -> String { "base" }
}
```

## Why Use Final?

- Communicates no subclassing
- Protects design intent
- Can help optimization
- Reduces override complexity

## Interview Levels

Junior:

`final` means a class cannot be subclassed.

Senior:

I use `final` by default for classes not designed for inheritance. Inheritance is an API commitment, so blocking it keeps behavior predictable.

## Quick Notes

- `final class` cannot be subclassed
- `final func` cannot be overridden
- Good default for app classes
- Inheritance should be intentional

## Interview Depth

Junior answer: `final` prevents subclassing or overriding.

Mid-level answer: Use `final` when a class or method is not designed for inheritance.

Senior answer: Inheritance is an API promise. Marking app classes `final` protects design intent, simplifies reasoning, and may help optimization by enabling static dispatch.

iOS use case:

```swift
final class LoginViewModel {
    func submit() { }
}
```

Common mistakes:

- Leaving classes open for accidental subclassing.
- Marking base framework classes final when subclassing is intended.
- Thinking `final` is only for performance.

Practice:

1. Mark service class final.
2. Explain why final improves design.
3. Explain when not to use final.
