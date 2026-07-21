# Day 33: SOLID Interview Guide

## One-Minute Senior Answer

SOLID principles help manage change. SRP keeps each type focused around one reason to change. OCP lets us add behavior through extension points rather than risky edits. LSP ensures implementations honor the same contract. ISP keeps protocols small so clients depend only on what they use. DIP keeps high-level policy independent from low-level details through abstractions and injection. In iOS, I apply SOLID to ViewModels, services, repositories, coordinators, SwiftUI components, and SDK boundaries, but I avoid ceremonial abstraction where simple code is clearer.

## Principle Quick Map

```text
SRP:
One reason to change.

OCP:
Extend behavior without repeatedly modifying stable code.

LSP:
Implementations must be substitutable.

ISP:
Small focused protocols/interfaces.

DIP:
Depend on abstractions, inject concrete details.
```

## Junior Questions

What is SOLID?

A set of five design principles for maintainable object-oriented code.

What is SRP?

One class/type should have one clear responsibility.

What is Dependency Inversion?

High-level code should depend on abstractions, not concrete low-level implementations.

## Mid-Level Questions

How does SOLID improve testability?

Focused types and injected dependencies make it easier to test behavior with fakes, stubs, and spies.

How does ISP apply to Swift protocols?

Instead of one giant protocol, create smaller protocols for specific capabilities.

How does OCP apply in iOS?

Use strategies, protocols, extensions, and dependency injection to add new behavior like payment methods or analytics providers without editing stable feature code.

## Senior Questions

How do you know SRP is violated?

I look for multiple reasons to change, unrelated dependencies, difficult tests, and files that require edits for networking, UI, persistence, analytics, and business rules together.

When is OCP overengineering?

When variation is speculative or a finite enum switch is clearer. I use OCP when the domain is genuinely open-ended or stable code is risky to edit.

How can mocks violate LSP?

If a mock returns impossible values, ignores validation, or handles errors differently from production, tests become misleading because the implementation is not substitutable.

How do you avoid protocol soup?

I create protocols around real boundaries: effects, module seams, testing needs, or multiple implementations. I avoid protocols for every tiny helper.

How do SOLID and SwiftUI fit together?

SwiftUI views should render state and send intents. Business rules, effects, navigation, and persistence should live behind focused models, services, repositories, or routers when complexity justifies it.

## Senior Artifact: SOLID Code Review Checklist

```text
Type under review:
SRP:
- How many reasons to change?
- Are responsibilities cohesive?

OCP:
- Is variation expected?
- Are extension points stable?

LSP:
- Do implementations honor contracts?
- Are test doubles realistic?

ISP:
- Do clients depend on unused methods?
- Are protocols cohesive?

DIP:
- Does policy depend on infrastructure?
- Are dependencies injected?

Decision:
Refactor now or leave simple?
```

## Real Scenario: Checkout

SOLID design:

- `CheckoutView`: renders UI.
- `CheckoutViewModel`: presentation state and user intents.
- `CheckoutUseCase`: payment/order workflow.
- `PaymentProcessing`: protocol for payment providers.
- `OrderRepository`: order persistence/network boundary.
- `AnalyticsTracking`: analytics abstraction.
- `CheckoutRouter`: navigation.

Principles:

- SRP: each type has clear job.
- OCP: add payment provider by adding processor.
- LSP: all payment processors follow same contract.
- ISP: checkout depends only on needed capabilities.
- DIP: ViewModel/use case depend on abstractions.

## Common Interview Traps

- Reciting definitions without examples.
- Saying every type needs a protocol.
- Saying SOLID means no modifications ever.
- Ignoring tradeoffs.
- Overusing inheritance in Swift examples.
- Not connecting SOLID to testing.
- Not explaining when not to use it.

## Strong Closing Answer

SOLID is useful when it makes change safer. I apply it where responsibilities are mixed, dependencies are concrete, protocols are too broad, implementations break contracts, or features need stable extension points. I avoid turning SOLID into needless indirection.

## Practice Prompts

1. Explain all five SOLID principles with iOS examples.
2. Refactor a login ViewModel using SOLID.
3. Design checkout using all five principles.
4. Explain enum vs protocol for OCP.
5. Explain how test doubles can violate LSP.
6. Review a giant protocol using ISP.
