# Day 30: Design Patterns Interview Guide

## One-Minute Senior Answer

iOS design patterns are tools for managing ownership, creation, communication, state, navigation, and external effects. I use dependency injection to make dependencies explicit, coordinators for navigation, repositories for data boundaries, facades for complex subsystems, services/use cases for effects and workflows, factories/assemblies for construction, adapters for SDK boundaries, strategies/policies for variable business rules, and state management patterns to prevent impossible UI states. I avoid applying patterns mechanically.

## Core Pattern Map

```text
Dependency Injection: make dependencies explicit
Coordinator: own navigation flow
Repository: hide data sources
Service Layer: external technical capability
Use Case: business workflow
Facade: simple API over complex subsystem
Singleton: one shared global instance, use carefully
Factory: create objects
Assembly: wire modules
Builder: construct complex values
Adapter: translate external interface
Strategy: swap algorithms
Policy: encode business rules
Observer/Publisher/Delegate: communicate events
State Management: own and mutate state predictably
```

## Junior Questions

What is dependency injection?

Passing dependencies into a type instead of creating them inside.

What is coordinator pattern?

A pattern that moves navigation flow out of views/view controllers.

What is repository pattern?

A pattern that hides data source details behind a domain-friendly interface.

What is singleton?

A single shared instance, usually accessed globally.

## Mid-Level Questions

Repository vs service?

A service performs a capability, often technical. A repository represents domain data access and may combine network, cache, persistence, and mapping.

Facade vs adapter?

Facade simplifies a complex subsystem. Adapter converts one interface into another.

Factory vs builder?

Factory creates objects. Builder constructs complex values step by step.

Delegate vs closure?

Closure is simple for one event. Delegate is better for multiple related callbacks and UIKit-style one-to-one communication.

## Senior Questions

How do you avoid overengineering with patterns?

I start with the problem: ownership, creation, navigation, communication, state, data, or side effects. I add a pattern only when it reduces real complexity or improves testability/modularity.

When is singleton acceptable?

For truly process-wide infrastructure or stateless entry points, but feature code should usually depend on protocols injected at boundaries. Mutable singletons need thread-safety review.

How do you design a service/use-case layer?

Services represent technical capabilities; use cases represent business workflows. ViewModels call use cases when one user action coordinates multiple services or business rules.

How do you choose state management?

I classify state by owner and lifetime: local view state, feature state, app/session state, shared state, persisted state, and derived state. Then I choose property wrappers, observable models, reducers, or repositories accordingly.

## Senior Artifact: Pattern Decision Matrix

```text
Problem:
Pattern considered:
Ownership issue:
Creation issue:
Navigation issue:
Data source issue:
Communication issue:
State issue:
Testing need:
Chosen pattern:
Why:
Tradeoff:
```

## Real Scenario: Checkout

Patterns:

- DI: inject payment/order/analytics dependencies.
- Use Case: coordinate payment and order creation.
- Repository: create and fetch orders.
- Coordinator/Router: show confirmation.
- State enum: editing/submitting/completed/failed.
- Facade: wrap payment SDK if complex.
- Strategy/Policy: retry/idempotency/payment validation.
- Adapter: hide vendor payment SDK.

Senior discussion:

Checkout is risky because payment side effects can duplicate. Patterns are not decoration here; they isolate risk.

## Common Interview Traps

- Saying singleton is always bad or always good.
- Using repository and service interchangeably without nuance.
- Applying coordinator to every tiny SwiftUI view.
- Creating protocols for everything.
- Not mentioning testability.
- Not discussing tradeoffs.
- Pattern names without examples.

## Strong Closing Answer

Patterns are vocabulary for design decisions. A senior engineer knows when they help, when they hurt, and how they affect testing, ownership, memory, concurrency, and maintainability.

## Practice Prompts

1. Design checkout using at least five patterns.
2. Refactor singleton analytics into DI.
3. Choose communication pattern for child-to-parent events.
4. Compare facade, adapter, and repository.
5. Explain state ownership for cart.
6. Write an architecture decision for using Coordinator.
