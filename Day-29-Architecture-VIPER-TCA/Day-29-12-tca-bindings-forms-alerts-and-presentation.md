# Day 29: TCA Bindings, Forms, Alerts, And Presentation

## Why Presentation Modeling Matters

Forms, sheets, alerts, dialogs, and navigation are common sources of messy state. TCA makes them explicit.

This helps you avoid:

- Many boolean flags
- Unclear dismissal behavior
- Untested validation
- Side effects hidden in views
- Alert state scattered across UI

## Form State

```swift
@ObservableState
struct State: Equatable {
    var email = ""
    var password = ""
    var isSubmitting = false
    var alert: AlertState<Action.Alert>?

    var canSubmit: Bool {
        email.contains("@") && password.count >= 8 && !isSubmitting
    }
}
```

## Binding Actions

Modern TCA supports binding patterns so fields can update state cleanly.

```swift
enum Action: BindableAction {
    case binding(BindingAction<State>)
    case signInButtonTapped
    case signInResponse(Result<UserSession, LoginError>)
    case alert(PresentationAction<Alert>)

    enum Alert: Equatable {
        case dismiss
    }
}
```

Reducer:

```swift
var body: some Reducer<State, Action> {
    BindingReducer()

    Reduce { state, action in
        switch action {
        case .binding:
            state.alert = nil
            return .none

        case .signInButtonTapped:
            guard state.canSubmit else { return .none }
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
            state.alert = AlertState {
                TextState("Sign In Failed")
            } actions: {
                ButtonState(role: .cancel, action: .dismiss) {
                    TextState("OK")
                }
            } message: {
                TextState("Check your email and password.")
            }
            return .none

        case .alert:
            return .none
        }
    }
}
```

## SwiftUI View

```swift
struct LoginView: View {
    @Bindable var store: StoreOf<LoginFeature>

    var body: some View {
        Form {
            TextField("Email", text: $store.email)
                .textInputAutocapitalization(.never)

            SecureField("Password", text: $store.password)

            Button("Sign In") {
                store.send(.signInButtonTapped)
            }
            .disabled(!store.canSubmit)
        }
        .alert($store.scope(state: \.alert, action: \.alert))
    }
}
```

Exact presentation syntax can evolve by TCA version, but the design remains: presentation is state.

## Sheets And Destinations

```swift
@Presents var destination: Destination.State?

enum Action {
    case editButtonTapped
    case destination(PresentationAction<Destination.Action>)
}

@Reducer
enum Destination {
    case editProfile(EditProfileFeature)
}
```

Reducer:

```swift
case .editButtonTapped:
    state.destination = .editProfile(EditProfileFeature.State(profile: state.profile))
    return .none
```

View:

```swift
.sheet(
    item: $store.scope(state: \.destination?.editProfile, action: \.destination.editProfile)
) { store in
    EditProfileView(store: store)
}
```

## Validation

Keep validation in state/reducer/domain logic.

```swift
var emailValidationMessage: String? {
    guard !email.isEmpty else { return nil }
    return email.contains("@") ? nil : "Enter a valid email."
}
```

Senior note: business-critical validation should also exist in domain/use-case layer, not only UI state.

## Common Mistakes

- Many alert booleans instead of modeled alert state.
- Form validation only in the view.
- Views directly running effects.
- Presentation state not tested.
- Alert actions ignored in tests.
- Bindings used for state that should be changed by explicit actions.

## Senior Artifact

```text
Artifact: TCA Presentation Review
Feature:
Form fields:
Bindings:
Validation:
Alert state:
Sheet/destination state:
Dismissal behavior:
Submit effect:
Tests:
```

## Interview Notes

Junior:

TCA can bind form fields to store state.

Mid-level:

Use binding actions for forms and model alerts/sheets as state.

Senior:

I model presentation explicitly so forms, alerts, sheets, validation, and dismissal are testable instead of hidden in SwiftUI view code.

## Practice

1. Build a login form with binding actions.
2. Add alert state for failed sign-in.
3. Add sheet destination for edit profile.
4. Test that invalid submit does not run an effect.
