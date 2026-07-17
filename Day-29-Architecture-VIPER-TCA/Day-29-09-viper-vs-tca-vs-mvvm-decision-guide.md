# Day 29: VIPER vs TCA vs MVVM Decision Guide

## Why Compare Architectures

Senior engineers do not choose architecture by trend. They choose based on:

- Product complexity
- Team experience
- Test requirements
- Codebase history
- UI framework
- Modularization needs
- Delivery speed
- Maintenance horizon

## MVVM

Best for:

- Many SwiftUI apps
- Medium complexity screens
- Async loading and validation
- Teams wanting simple testable separation

Strengths:

- Familiar.
- Flexible.
- Low ceremony.
- Works with SwiftUI and UIKit.
- Easy to introduce gradually.

Weaknesses:

- Can create massive ViewModels.
- Navigation ownership can be unclear.
- Side effects can become scattered.
- Testing discipline varies.

## VIPER

Best for:

- Large UIKit apps
- Enterprise teams
- Strict module boundaries
- Complex business use cases
- Navigation-heavy flows

Strengths:

- Clear responsibilities.
- Strong separation.
- Testable use cases.
- Navigation isolated.
- Good for team-scale modules.

Weaknesses:

- Boilerplate.
- Many files/protocols.
- Can feel heavy in SwiftUI.
- Easy to do ceremonially.

## TCA

Best for:

- Complex state machines
- SwiftUI apps with many effects
- High test requirements
- Apps needing reducer composition
- Teams comfortable with functional/reducer architecture

Strengths:

- Explicit state/actions/effects.
- Excellent testing.
- Great composition story.
- Navigation as state.
- Strong dependency control.

Weaknesses:

- Learning curve.
- Library dependency.
- Ceremony for simple features.
- Reducers can grow large if unmanaged.

## Quick Decision Table

```text
Small static screen:
MVVM or plain SwiftUI

Medium async screen:
MVVM

Large UIKit feature:
VIPER or MVVM+Coordinator

Complex SwiftUI state machine:
TCA

Enterprise module with strict teams:
VIPER

Highly testable reducer-driven app:
TCA

Prototype:
Plain SwiftUI or light MVVM
```

## Example: Login

Simple login:

- MVVM is usually enough.

Enterprise login with multi-step policy, 2FA, compliance, routing:

- VIPER or TCA can help.

Reducer-heavy login with timers, resend code, navigation, side effects:

- TCA is strong.

## Example: Checkout

Checkout has:

- Form state
- Validation
- Payment side effect
- Order creation
- Retry risk
- Navigation
- Analytics

Good options:

- MVVM + use case + router
- TCA for strong effect/state testing
- VIPER for UIKit enterprise module boundaries

## Migration Thinking

Do not rewrite architecture casually.

Good migration:

1. Pick one feature.
2. Keep behavior tests.
3. Move boundaries gradually.
4. Avoid mixing patterns inside one small feature.
5. Document the decision.

## Architecture Decision Record

```text
ADR: Use TCA for Search
Context:
Search has debounce, cancellation, pagination, and deep link selection.

Decision:
Use TCA for explicit actions/effects and TestStore coverage.

Alternatives:
MVVM with manual cancellation
VIPER module

Consequences:
Higher learning curve, stronger tests, consistent state flow.
```

## Senior Artifact

```text
Artifact: Architecture Selection Matrix
Feature:
UI framework:
Complexity:
State machine:
Side effects:
Navigation:
Testing level:
Team skill:
Timeline:
Recommended pattern:
Why:
Risks:
```

## Common Mistakes

- Choosing the most complex pattern by default.
- Mixing all patterns randomly.
- No tests despite choosing a testable architecture.
- Rewriting architecture without business value.
- Ignoring team familiarity.
- Treating folder structure as architecture.

## Interview Notes

Junior:

MVVM, VIPER, and TCA are ways to organize app code.

Mid-level:

MVVM is flexible, VIPER is strict and modular, TCA is state/action/effect driven.

Senior:

I choose architecture by problem shape. MVVM is often a pragmatic default, VIPER fits strict UIKit modules, and TCA fits complex state/effect-heavy apps with high test requirements.

## Practice

1. Choose architecture for a simple profile screen.
2. Choose architecture for a complex checkout flow.
3. Write an ADR for using TCA in search.
4. Explain why architecture migration should be feature-by-feature.
