# Day 29: VIPER And TCA Interview Guide And Senior Artifacts

## One-Minute Senior Answer

VIPER and TCA both try to make complex iOS apps more predictable, but they do it differently. VIPER separates a module into View, Interactor, Presenter, Entity, and Router, which is useful for strict UIKit module boundaries and testable use cases. TCA models features with State, Action, Reducer, Effect, Store, and Dependencies, which is powerful for SwiftUI apps with complex state, side effects, navigation, and exhaustive tests. I choose between them based on complexity, team skill, test needs, UI framework, and maintenance cost.

## VIPER Core Answers

What is VIPER?

VIPER is an iOS architecture based on Clean Architecture that separates View, Interactor, Presenter, Entity, and Router.

What does Presenter do?

It receives user events, requests work from Interactor, formats display models, updates View, and asks Router to navigate.

What does Interactor do?

It contains business/use-case logic and talks to repositories or services.

What does Router do?

It builds modules and performs navigation.

When use VIPER?

Large UIKit apps, strict module boundaries, complex business flows, and teams that value testable separation over boilerplate cost.

## TCA Core Answers

What is TCA?

The Composable Architecture is a Swift architecture library based on explicit State, Action, Reducer, Effect, Store, Dependencies, and composition.

What is a reducer?

A reducer receives state and an action, mutates state, and returns effects.

What is an effect?

An effect is work outside the reducer, such as network, timers, persistence, analytics, or location, that can send actions back.

Why is TCA testable?

`TestStore` lets you send actions, assert state changes, control dependencies/time, and assert received effect actions.

When use TCA?

Complex SwiftUI apps with many state transitions, side effects, navigation paths, composition needs, and high testing expectations.

## Senior Questions

VIPER vs MVVM?

MVVM is lighter and often enough for SwiftUI or medium-complexity screens. VIPER is stricter, more modular, and more boilerplate-heavy, often fitting large UIKit modules.

TCA vs MVVM?

MVVM is simpler and flexible. TCA is more explicit and testable for state/effect-heavy flows, but has learning curve and ceremony.

VIPER vs TCA?

VIPER separates responsibilities by object roles. TCA centralizes behavior around state/actions/reducers/effects. VIPER is usually more UIKit/module oriented; TCA is usually more SwiftUI/state-machine oriented.

How do you avoid overengineering?

I start with the simplest architecture that makes ownership, testing, and change safe. I add stricter patterns only when complexity justifies them.

## Senior Artifact: VIPER Module Review

```text
Module:
View responsibility:
Presenter responsibility:
Interactor use cases:
Entities:
Router transitions:
Assembly:
Protocols:
Memory ownership:
Presenter tests:
Interactor tests:
Navigation tests:
Boilerplate justified:
```

## Senior Artifact: TCA Feature Review

```text
Feature:
State:
Actions:
Reducer:
Effects:
Dependencies:
Cancellation:
Navigation:
Composition:
TestStore coverage:
Shared state:
Reducer size:
Tradeoffs:
```

## Senior Artifact: Architecture Decision

```text
Feature:
Pattern considered:
- MVVM
- VIPER
- TCA
Complexity:
State/effects:
Navigation:
Team skill:
Testing need:
Timeline:
Decision:
Risks:
Review trigger:
```

## Real Scenario: Search Feature

Requirements:

- Debounced search
- Cancellation
- Pagination
- Recent searches
- Deep link to result
- Error recovery
- Analytics
- Tests

MVVM:

- Good if team wants lower ceremony.
- Needs manual cancellation and tests.

VIPER:

- Good in UIKit enterprise app.
- Presenter handles UI events, Interactor handles search use case, Router handles result navigation.

TCA:

- Strong fit because debounce, cancellation, pagination, effects, and navigation can be modeled and tested explicitly.

## Common Interview Traps

- Saying VIPER is always better because it has more layers.
- Saying TCA is only for SwiftUI.
- Not knowing what Interactor does.
- Not knowing what an effect is.
- Ignoring testing.
- Ignoring boilerplate and team skill.
- Choosing architecture without context.

## Strong Closing Answer

Architecture is a tool for managing change. VIPER gives strict object boundaries; TCA gives explicit reducer-based state and effects. I pick the pattern that makes the feature easiest to reason about, test, and evolve, while keeping ceremony proportional to complexity.

## Practice Prompts

1. Explain VIPER in two minutes.
2. Explain TCA in two minutes.
3. Compare VIPER and TCA for checkout.
4. Design a VIPER product details module.
5. Design a TCA search feature.
6. Write an architecture decision record.
7. Explain how each pattern handles navigation.
8. Explain how each pattern handles testing.
