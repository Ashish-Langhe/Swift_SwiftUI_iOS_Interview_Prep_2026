# Day 12: Protocol Declaration

## Core Idea

A protocol defines a contract.

```swift
protocol IdentifiableUser {
    var id: String { get }
}
```

Types conform by providing required members.

## Real iOS Use Cases

- Services
- Delegates
- Testable abstractions
- View model contracts

## Modern Swift 6.x Notes

Protocols interact deeply with `any`, `some`, associated types, and concurrency. Protocols used across tasks may need `Sendable`.

## Interview Levels

Junior: A protocol defines requirements.

Senior: Protocols define capabilities and abstraction boundaries. They improve testability but should not be created without a real abstraction need.

## Quick Notes

- Declared with `protocol`.
- Defines required properties/methods.
- Conformance supplies implementation.

## Interview Depth

Junior answer: A protocol defines requirements that other types can follow.

Mid-level answer: Protocols are useful for abstraction, delegates, services, and testability.

Senior answer: A protocol should represent a real capability or boundary. Avoid creating protocols only for ceremony when there is one concrete type and no testing or substitution need.

iOS use case:

```swift
protocol AuthServicing {
    func login(email: String, password: String) async throws -> User
}
```

Common mistakes: huge protocols, vague names, protocol for every class.

Practice: declare service protocol, explain capability, decide if protocol is needed.
