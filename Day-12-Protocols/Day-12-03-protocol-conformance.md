# Day 12: Protocol Conformance

## Core Idea

A type conforms to a protocol by implementing requirements.

```swift
struct User: IdentifiableUser {
    let id: String
}
```

## Real iOS Use Cases

```swift
protocol PaymentService {
    func pay(amount: Decimal) async throws
}

final class StripePaymentService: PaymentService {
    func pay(amount: Decimal) async throws { }
}
```

## Interview Levels

Junior: Conformance means a type follows a protocol.

Senior: Protocol conformance is how Swift models capabilities. Use it to decouple callers from concrete implementations, especially for testing and dependency injection.

## Quick Notes

- Add `: ProtocolName`.
- Implement all requirements.
- Can conform in extensions.

## Interview Depth

Junior answer: Conformance means a type implements a protocol.

Mid-level answer: A type can conform in its declaration or in an extension. Extensions help organize conformances.

Senior answer: Conformance should be meaningful. Retroactive conformances can be useful, but be cautious when conforming types you do not own to protocols you do not own.

iOS use case:

```swift
extension RemoteAuthService: AuthServicing { }
```

Common mistakes: empty conformance without actual behavior, conflicting conformances, putting test-only conformance in production code.

Practice: conform a service to protocol, conform model to Identifiable, organize in extension.
