# Day 33: Open/Closed Principle

## What OCP Means

Open/Closed Principle says:

```text
Software entities should be open for extension but closed for modification.
```

In practical iOS terms:

- Add new payment methods without rewriting checkout.
- Add new analytics providers without editing every feature.
- Add new sorting rules without changing core list logic.
- Add new screen routes without fragile conditionals everywhere.

OCP does not mean "never edit existing code." It means stable code should not need risky edits for every new variation.

## Bad Example: Payment Switch Everywhere

```swift
enum PaymentMethod {
    case card
    case applePay
    case paypal
}

final class CheckoutService {
    func pay(method: PaymentMethod, amount: Decimal) async throws {
        switch method {
        case .card:
            try await chargeCard(amount)
        case .applePay:
            try await chargeApplePay(amount)
        case .paypal:
            try await chargePayPal(amount)
        }
    }
}
```

This is acceptable when payment methods are few and stable. It becomes painful when adding methods requires editing many switches across validation, UI, analytics, and payment.

## OCP Refactor With Strategy

```swift
protocol PaymentProcessor {
    var methodID: String { get }
    func canPay(amount: Decimal) -> Bool
    func pay(amount: Decimal) async throws -> PaymentResult
}
```

Implementations:

```swift
struct CardPaymentProcessor: PaymentProcessor {
    let methodID = "card"

    func canPay(amount: Decimal) -> Bool {
        amount > 0
    }

    func pay(amount: Decimal) async throws -> PaymentResult {
        // card payment
    }
}

struct ApplePayProcessor: PaymentProcessor {
    let methodID = "applePay"

    func canPay(amount: Decimal) -> Bool {
        amount > 0 && ApplePayCapability.isAvailable
    }

    func pay(amount: Decimal) async throws -> PaymentResult {
        // Apple Pay
    }
}
```

Checkout:

```swift
struct CheckoutUseCase {
    let processors: [String: PaymentProcessor]

    func pay(methodID: String, amount: Decimal) async throws -> PaymentResult {
        guard let processor = processors[methodID] else {
            throw PaymentError.unsupportedMethod
        }

        guard processor.canPay(amount: amount) else {
            throw PaymentError.unavailable
        }

        return try await processor.pay(amount: amount)
    }
}
```

Adding a new payment method adds a new processor.

## OCP With Protocol Extensions

```swift
protocol AnalyticsEvent {
    var name: String { get }
    var parameters: [String: String] { get }
}

extension AnalyticsEvent {
    var parameters: [String: String] { [:] }
}
```

New events extend behavior:

```swift
struct CheckoutStartedEvent: AnalyticsEvent {
    let name = "checkout_started"
    let cartID: String

    var parameters: [String: String] {
        ["cart_id": cartID]
    }
}
```

Analytics client does not need to know every event type.

## OCP With SwiftUI Components

Bad:

```swift
struct StatusBadge: View {
    let status: String

    var color: Color {
        if status == "paid" { .green }
        else if status == "failed" { .red }
        else { .gray }
    }
}
```

Better:

```swift
protocol BadgeDisplayable {
    var title: String { get }
    var tint: Color { get }
}

struct StatusBadge<Status: BadgeDisplayable>: View {
    let status: Status

    var body: some View {
        Text(status.title)
            .foregroundStyle(status.tint)
    }
}
```

Different domains can provide badge display data without editing `StatusBadge`.

## OCP With Repositories

```swift
protocol ProductRepository {
    func products() async throws -> [Product]
}

struct NetworkProductRepository: ProductRepository {}
struct CachedProductRepository: ProductRepository {}
struct PreviewProductRepository: ProductRepository {}
```

ViewModel is closed to repository implementation changes.

## When To Use OCP

Use when:

- Variation is expected.
- Switch statements are growing.
- New types are added more often than core behavior changes.
- Multiple teams add implementations.
- Stable code is risky to edit.

Do not force when:

- Variation is speculative.
- A simple enum switch is clearer.
- There are only two stable cases.
- Abstraction hides important business rules.

## Senior Decision: Enum vs Protocol

Use enum when:

- Cases are finite and known.
- Exhaustive switching is valuable.
- Behavior belongs centrally.

Use protocol/strategy when:

- Cases are open-ended.
- Third parties or modules add behavior.
- You need injection/testing.
- You want to avoid modifying stable code.

## Common Mistakes

- Creating abstractions for imaginary future cases.
- Hiding all logic behind protocols too early.
- Replacing clear enum switches with scattered classes.
- Breaking discoverability.
- OCP applied without tests.
- Assuming all modification is bad.

## Senior iOS Engineer Artifact

```text
Artifact: OCP Extension Review
Behavior:
Known variants:
Expected future variants:
Current switch/edit points:
Risk of modifying stable code:
Enum or protocol:
Extension mechanism:
Tests:
```

## Interview Notes

Junior:

OCP means code should allow new behavior without constantly changing existing code.

Mid-level:

Use protocols, strategies, dependency injection, and extensions to add behavior safely.

Senior:

I apply OCP only where variation is real. I choose enums for closed domains and protocols/strategies for open-ended domains, keeping extension points testable and understandable.

## Practice

1. Refactor payment switch into payment processors.
2. Decide enum vs protocol for order status.
3. Add a new analytics event without changing analytics client.
4. Explain when OCP creates overengineering.
