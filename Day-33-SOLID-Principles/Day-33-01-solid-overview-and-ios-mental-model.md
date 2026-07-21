# Day 33: SOLID Overview And iOS Mental Model

## What SOLID Means

SOLID is a set of five object-oriented design principles:

- Single Responsibility Principle
- Open/Closed Principle
- Liskov Substitution Principle
- Interface Segregation Principle
- Dependency Inversion Principle

These principles help code become:

- Easier to change
- Easier to test
- Easier to reuse
- Easier to review
- Less coupled
- More predictable

SOLID is not about making more files. It is about controlling reasons for change.

## SOLID In One Line Each

```text
SRP:
One type should have one clear reason to change.

OCP:
Add new behavior by extending code, not constantly editing stable code.

LSP:
Subtypes or conforming implementations should be safely replaceable.

ISP:
Prefer small focused interfaces over large do-everything protocols.

DIP:
High-level policy should depend on abstractions, not concrete details.
```

## Why iOS Engineers Need SOLID

iOS apps usually have:

- Views
- ViewModels
- Services
- Repositories
- Coordinators
- Persistence
- Networking
- Analytics
- Feature flags
- Design-system components
- Async workflows

Without SOLID thinking, these become tightly coupled.

Common symptoms:

- ViewModel does networking, formatting, analytics, navigation, and persistence.
- Singleton managers spread everywhere.
- One protocol has twenty unrelated methods.
- Adding a payment method requires editing many files.
- Tests need real network because dependencies are hidden.
- Subclass override breaks parent assumptions.

## SOLID Is About Change

Every principle asks a change question.

```text
SRP:
How many reasons does this type have to change?

OCP:
Can I add a new case without editing fragile stable logic?

LSP:
Can I replace this implementation without surprising callers?

ISP:
Does this client depend on methods it never uses?

DIP:
Does policy depend on concrete infrastructure?
```

## Simple Bad Example

```swift
@MainActor
final class CheckoutViewModel {
    var state: CheckoutState = .editing

    func submit(cart: Cart) async {
        state = .submitting

        do {
            let token = try KeychainManager.shared.token()
            let payment = try await URLSession.shared.charge(cart, token: token)
            try await AnalyticsManager.shared.track("checkout_success")
            try FileManager.default.save(order: payment.order)
            state = .completed(payment.order.id)
        } catch {
            state = .failed("Checkout failed.")
        }
    }
}
```

Problems:

- ViewModel owns too many responsibilities.
- Concrete singletons are hidden dependencies.
- Hard to test.
- Hard to reuse payment workflow.
- Persistence and analytics are tangled with presentation.

## Better Direction

```swift
protocol CheckoutUseCase {
    func execute(cart: Cart) async throws -> Order
}

@MainActor
final class CheckoutViewModel {
    var state: CheckoutState = .editing

    private let checkoutUseCase: CheckoutUseCase

    init(checkoutUseCase: CheckoutUseCase) {
        self.checkoutUseCase = checkoutUseCase
    }

    func submit(cart: Cart) async {
        state = .submitting

        do {
            let order = try await checkoutUseCase.execute(cart: cart)
            state = .completed(order.id)
        } catch {
            state = .failed("Checkout failed.")
        }
    }
}
```

The ViewModel now owns presentation state. The use case owns checkout workflow.

## SOLID And Reusability

SOLID improves reuse by reducing accidental coupling.

Reusable code tends to have:

- Focused responsibility
- Small interface
- Injected dependencies
- Stable contracts
- Replaceable implementations
- Clear invariants

But reuse is not always the goal. Sometimes clarity for one feature matters more than generalized abstraction.

Senior rule:

```text
Reuse should emerge from real repetition or stable boundaries, not speculation.
```

## SOLID And Testing

SOLID code is usually easier to test because:

- Smaller types have fewer setup needs.
- Protocol boundaries allow fakes.
- Business rules move out of views.
- Effects are injected.
- Interfaces are focused.

Example:

```swift
struct StubCheckoutUseCase: CheckoutUseCase {
    let result: Result<Order, Error>

    func execute(cart: Cart) async throws -> Order {
        try result.get()
    }
}
```

## SOLID With Swift

Swift changes how SOLID feels:

- Protocols are lightweight.
- Value types make composition natural.
- Extensions support open/closed design.
- Generics can reduce coupling.
- Actors affect dependency and state boundaries.
- SwiftUI encourages state-driven views.

SOLID should be applied in a Swifty way, not copied blindly from Java examples.

## Common Mistakes

- Treating SOLID as mandatory protocol-per-class design.
- Splitting code into many tiny files without clearer ownership.
- Overusing inheritance in Swift.
- Applying OCP by hiding all logic behind abstractions too early.
- Using DIP to create meaningless protocols.
- Ignoring value types and composition.

## Senior iOS Engineer Artifact

```text
Artifact: SOLID Architecture Review
Feature:
Types with many reasons to change:
Concrete dependencies:
Large protocols:
Subclass/override risks:
Extension points:
Reusable boundaries:
Testing blockers:
Suggested refactor:
Tradeoffs:
```

## Interview Notes

Junior:

SOLID principles help organize code so it is easier to maintain.

Mid-level:

SOLID reduces coupling and improves testability by separating responsibilities and depending on abstractions.

Senior:

I use SOLID as a decision lens. I do not create abstractions automatically. I look for actual reasons for change, unstable dependencies, test friction, reuse pressure, and boundaries where extension is safer than modification.

## Practice

1. Identify SOLID violations in a large ViewModel.
2. Explain each principle in one sentence.
3. Refactor a concrete service dependency into injection.
4. Decide when SOLID refactoring is overengineering.
