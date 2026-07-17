# Day 29: TCA Senior Interview Deep Dive

## Senior-Level TCA Summary

TCA is a reducer-based architecture that models a feature as explicit state, actions, effects, dependencies, and tests. Its strength is not just organization; it is making behavior exhaustively testable. I use it when feature complexity, side effects, navigation, and test requirements justify the learning curve and ceremony.

## Two-Minute Explanation

In TCA, the view does not mutate state directly. The view sends actions to a store. The store runs a reducer. The reducer mutates state and returns effects. Effects perform async or external work and send actions back. Dependencies are injected through TCA's dependency system, so tests can override networking, clocks, UUIDs, persistence, and analytics. Complex apps are built by composing small reducers.

## Senior Question: Why TCA Over MVVM?

Use TCA when:

- State transitions are complex.
- Effects need cancellation.
- Navigation should be testable.
- Team wants consistency.
- High test coverage matters.
- Features compose deeply.

Use MVVM when:

- Feature is simpler.
- Team wants less ceremony.
- ViewModel tests are enough.
- Library dependency is not desired.

Strong answer:

```text
I do not choose TCA because it is trendy. I choose it when explicit state/action/effect modeling and exhaustive tests reduce risk more than the framework cost adds friction.
```

## Senior Question: What Makes A Good Action?

Good actions represent meaningful events:

```swift
case signInButtonTapped
case emailChanged(String)
case signInResponse(Result<UserSession, LoginError>)
case alert(PresentationAction<Alert>)
```

Bad actions are vague:

```swift
case update
case loaded
case handle
```

Actions should tell a story in tests.

## Senior Question: How Do You Handle Effects?

Effects should:

- Be returned from reducers.
- Use dependencies.
- Send response actions.
- Be cancellable when long-running.
- Avoid hidden side effects in views.
- Be tested with dependency overrides.

Example:

```swift
return .run { send in
    await send(.response(Result { try await client.load() }))
}
```

## Senior Question: How Do You Handle Navigation?

Navigation is state:

- Optional/enum state for tree-based presentation.
- Stack state for push navigation.
- Presentation actions for sheets/alerts.
- Parent owns child navigation when appropriate.

This enables deep links and tests.

## Senior Question: How Do You Avoid Massive Reducers?

Split by feature boundaries:

- Child reducers
- Destination reducers
- Reducer composition
- Smaller state domains
- Delegate actions

If reducer action enum becomes a junk drawer, feature boundaries are probably wrong.

## Senior Question: What Are TCA Tradeoffs?

Costs:

- Learning curve
- Library dependency
- More explicit code
- Test strictness
- Version migration work

Benefits:

- Predictable state flow
- Strong tests
- Dependency control
- Composition
- Navigation as data
- Effect cancellation

## Senior Artifact: TCA Interview Answer Template

```text
Problem:
Why TCA fits:
State:
Actions:
Effects:
Dependencies:
Navigation:
Tests:
Tradeoffs:
Alternative:
```

## Example Answer: Checkout

Checkout has form validation, payment effects, order creation, analytics, retry risk, navigation, and error recovery.

I would model:

- `State`: draft, validation, submitting, alert, destination.
- `Action`: field changes, submit tapped, payment response, alert actions.
- `Dependencies`: payment client, order client, analytics, UUID/date.
- `Effects`: submit payment, create order, track success.
- `Navigation`: confirmation destination or route state.
- `Tests`: invalid submit, success flow, payment failure, duplicate submit prevention, analytics event, navigation.

## Common Senior Traps

- Saying TCA means no bugs.
- Ignoring dependency/version cost.
- Not knowing testing story.
- Putting side effects in views.
- Massive reducers.
- Actions named poorly.
- No cancellation.
- Treating TCA as only SwiftUI.

## Strong Closing Answer

TCA is strongest when a feature's behavior is complex enough that explicit modeling pays off. I use it to make state, effects, dependencies, and navigation testable. I avoid it where lightweight SwiftUI or MVVM is clearer.

## Practice Prompts

1. Explain TCA to a UIKit engineer.
2. Explain TCA to an MVVM team.
3. Design TCA for checkout.
4. Design TCA for search with debounce.
5. Explain dependency overrides.
6. Explain cancellation.
7. Explain navigation as state.
8. Explain when not to use TCA.
