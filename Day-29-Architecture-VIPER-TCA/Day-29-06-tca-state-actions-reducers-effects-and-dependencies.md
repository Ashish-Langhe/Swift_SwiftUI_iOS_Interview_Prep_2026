# Day 29: TCA State, Actions, Reducers, Effects, And Dependencies

## State Design

TCA state should include data needed for logic and rendering.

```swift
@ObservableState
struct State: Equatable {
    var email = ""
    var password = ""
    var isSubmitting = false
    var errorMessage: String?

    var canSubmit: Bool {
        email.contains("@") && password.count >= 8 && !isSubmitting
    }
}
```

Do not store cheap derived state unless needed.

## Action Design

Actions should describe what happened.

```swift
enum Action {
    case emailChanged(String)
    case passwordChanged(String)
    case signInButtonTapped
    case signInResponse(Result<UserSession, SignInError>)
}
```

Good actions are specific and testable.

Weak:

```swift
case update
case doThing
case handleResult
```

## Reducer Design

Reducers should be deterministic for a given action and state.

```swift
Reduce { state, action in
    switch action {
    case .emailChanged(let email):
        state.email = email
        state.errorMessage = nil
        return .none

    case .passwordChanged(let password):
        state.password = password
        state.errorMessage = nil
        return .none

    case .signInButtonTapped:
        guard state.canSubmit else {
            return .none
        }

        state.isSubmitting = true
        let email = state.email
        let password = state.password

        return .run { send in
            await send(.signInResponse(
                Result { try await authClient.signIn(email, password) }
            ))
        }

    case .signInResponse(.success):
        state.isSubmitting = false
        return .none

    case .signInResponse(.failure):
        state.isSubmitting = false
        state.errorMessage = "Invalid email or password."
        return .none
    }
}
```

## Effects

Effects represent work outside the reducer:

- Network
- Database
- Timers
- Location
- Notifications
- Analytics
- File I/O

Effects can send actions back.

```swift
return .run { send in
    let products = try await productClient.products()
    await send(.productsResponse(.success(products)))
}
```

## Dependencies

TCA dependencies make effects testable.

```swift
struct AuthClient {
    var signIn: (String, String) async throws -> UserSession
}
```

Reducer:

```swift
@Dependency(\.authClient) var authClient
```

Test can override:

```swift
let store = TestStore(initialState: LoginFeature.State()) {
    LoginFeature()
} withDependencies: {
    $0.authClient.signIn = { _, _ in .sample }
}
```

## Cancellation

Long-running effects should be cancellable.

```swift
enum CancelID {
    case search
}

case .queryChanged(let query):
    state.query = query

    return .run { send in
        try await clock.sleep(for: .milliseconds(300))
        let results = try await searchClient.search(query)
        await send(.searchResponse(.success(results)))
    }
    .cancellable(id: CancelID.search, cancelInFlight: true)
```

This is essential for search and live data.

## Error Modeling

Prefer domain/display errors over raw errors.

```swift
enum LoginError: Error, Equatable {
    case invalidCredentials
    case offline
    case unknown
}
```

State:

```swift
var alert: AlertState<Action.Alert>?
```

## Senior Artifact

```text
Artifact: TCA Reducer Review
Feature:
State shape:
Actions:
Derived state:
Effects:
Dependencies:
Cancellation IDs:
Error model:
Tests:
Reducer size:
Composition plan:
```

## Common Mistakes

- State too broad.
- Actions too generic.
- Effects not cancellable.
- Real dependencies in tests.
- Reducer performing formatting-heavy work repeatedly.
- Missing response actions.
- Side effects hidden in views.

## Interview Notes

Junior:

State stores data, actions describe events, reducers change state, effects run async work.

Mid-level:

Dependencies make effects testable and cancellation controls long-running work.

Senior:

I design TCA features with explicit state, precise actions, deterministic reducers, controlled dependencies, cancellable effects, and tests proving the full action/effect flow.

## Practice

1. Model login state and actions.
2. Add a sign-in effect.
3. Add dependency override for tests.
4. Add cancellable debounced search.
