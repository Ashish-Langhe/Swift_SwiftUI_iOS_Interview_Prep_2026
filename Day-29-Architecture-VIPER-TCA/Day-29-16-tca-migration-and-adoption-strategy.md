# Day 29: TCA Migration And Adoption Strategy

## Why Migration Strategy Matters

TCA is powerful, but adopting it across an app is a major architectural decision.

Bad adoption:

- Rewrite everything.
- No tests.
- Team does not understand reducers.
- Existing features lose behavior.
- TCA and old patterns mix randomly.

Good adoption:

- Start with one feature.
- Keep boundaries clear.
- Add tests.
- Document decisions.
- Teach the team.

## Good First TCA Features

Choose a feature with:

- Clear state machine
- Async effects
- Measurable test value
- Limited scope
- Few legacy dependencies

Good candidates:

- Search
- Login
- Settings form
- Onboarding flow
- Product details

Poor first candidates:

- Entire app root
- Highly coupled legacy dashboard
- Feature under urgent deadline
- Huge checkout rewrite without tests

## Migration From MVVM

MVVM:

```text
ViewModel state -> TCA State
ViewModel methods -> TCA Actions
Injected services -> TCA Dependencies
Async methods -> TCA Effects
ViewModel tests -> TestStore tests
```

Example:

```swift
func signIn() async
```

becomes:

```swift
case signInButtonTapped
case signInResponse(Result<UserSession, LoginError>)
```

## Migration From VIPER

VIPER:

```text
Presenter view events -> Actions
Presenter display state -> State
Interactor use cases -> Dependencies or reducers/effects
Router -> Navigation state
Presenter tests -> TestStore tests
```

The biggest mental shift: TCA centralizes behavior in reducer/state instead of object roles.

## Incremental Adoption

Use TCA for one feature while the rest of app remains MVVM/UIKit.

SwiftUI entry:

```swift
TCAFeatureView(
    store: Store(initialState: Feature.State()) {
        Feature()
    }
)
```

UIKit can host SwiftUI:

```swift
let view = FeatureView(store: store)
let controller = UIHostingController(rootView: view)
```

Or UIKit can observe a TCA store directly if needed.

## Team Training

TCA needs shared vocabulary:

- State
- Action
- Reducer
- Effect
- Dependency
- Store
- TestStore
- Scope
- Presentation

Without shared vocabulary, reducers become confusing.

## Upgrade Strategy

TCA evolves. Recent releases have introduced deprecations preparing for future major versions.

Senior approach:

- Pin versions deliberately.
- Read release notes.
- Upgrade in small steps.
- Keep tests green.
- Avoid deprecated APIs in new code.
- Budget migration time.

## Architecture Decision Record

```text
ADR: Adopt TCA For Search
Context:
Search has debounce, cancellation, pagination, and navigation.

Decision:
Implement SearchFeature in TCA as pilot.

Success criteria:
Reducer tests cover debounce, success, failure, cancellation, navigation.

Risks:
Team learning curve.
Library upgrade maintenance.
Integration with existing MVVM navigation.
```

## Common Mistakes

- Big-bang rewrite.
- No pilot feature.
- No TestStore coverage.
- TCA only in name, side effects still in views.
- Reducers imitate ViewModels poorly.
- Ignoring release migrations.
- No team agreement on patterns.

## Senior Artifact

```text
Artifact: TCA Adoption Plan
Pilot feature:
Why TCA:
Current pattern:
Migration map:
Dependencies:
Testing plan:
Training needs:
Version policy:
Success criteria:
Rollback plan:
```

## Interview Notes

Junior:

TCA can be adopted one feature at a time.

Mid-level:

MVVM ViewModel methods become actions/effects, and services become dependencies.

Senior:

I adopt TCA incrementally with a pilot feature, tests, team training, version strategy, and clear success criteria. I avoid big rewrites and architecture churn without product value.

## Practice

1. Map a login ViewModel into TCA state/actions/effects.
2. Write an ADR for TCA adoption.
3. Choose a good pilot feature.
4. Explain TCA migration risks to a team lead.
