# Day 12: Protocol-Oriented Programming

## Core Idea

Protocol-oriented programming models behavior as protocols and composes capabilities.

```swift
protocol Cacheable {
    var cacheKey: String { get }
}
```

## Real iOS Use Cases

- Service abstractions
- Mocking dependencies
- Delegates
- Feature capabilities

## Interview Levels

Junior: It means using protocols to define shared behavior.

Senior: Protocol-oriented programming is useful, but not every abstraction needs a protocol. I introduce protocols when there are multiple implementations, testing boundaries, or clear capability modeling.

## Quick Notes

- Compose behavior.
- Helps testing.
- Avoid protocol-for-everything.
- Prefer small focused protocols.

## Interview Depth

Junior answer: Protocol-oriented programming means using protocols to define shared behavior.

Mid-level answer: It helps with testability, dependency injection, and composing capabilities without inheritance.

Senior answer: Protocol-oriented programming is a tool, not a rule. Use protocols where they create useful boundaries. Avoid abstracting every concrete type prematurely.

iOS use case:

```swift
protocol TokenProviding {
    var token: String? { get }
}
```

Common mistakes: massive protocols, one-protocol-per-class habit, hiding simple code behind unnecessary abstraction.

Practice: create mockable service protocol, split broad protocol, explain composition over inheritance.
