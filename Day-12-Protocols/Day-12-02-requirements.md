# Day 12: Requirements

## Core Idea

Protocols can require properties, methods, initializers, and associated types.

```swift
protocol LoginServicing {
    func login(email: String, password: String) async throws -> User
}
```

Property requirement:

```swift
protocol Titled {
    var title: String { get }
}
```

## Interview Levels

Junior: Requirements are things conforming types must implement.

Senior: Protocol requirements should be minimal and behavior-focused. Large protocols become hard to mock and conform to.

## Quick Notes

- `{ get }` read-only requirement.
- `{ get set }` readable and writable.
- Methods can be async/throws.
- Keep protocols focused.

## Interview Depth

Junior answer: Requirements are properties or methods a conforming type must implement.

Mid-level answer: Protocols can require read-only properties, get-set properties, methods, initializers, and associated types.

Senior answer: Requirements should be minimal. A broad protocol makes mocking, conformance, and evolution harder.

iOS use case:

```swift
protocol ProfileLoading {
    var isLoading: Bool { get }
    func loadProfile() async throws
}
```

Common mistakes: requiring setters unnecessarily, mixing unrelated responsibilities, leaking implementation details.

Practice: define read-only requirement, async throws method, split a large protocol.
