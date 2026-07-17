# Day 28: MVVM Interview Guide And Senior Artifacts

## One-Minute Senior Answer

MVVM separates UI rendering from presentation behavior. The View renders state and forwards user intents. The ViewModel owns presentation state, validation, formatting, async transitions, and calls injected dependencies. The Model layer owns domain data, repositories, services, and business rules. At senior level, I use MVVM selectively, keep ownership clear, avoid massive ViewModels, inject dependencies, model state explicitly, test transitions, and choose navigation ownership deliberately.

## Core Mental Models

- View renders and sends intents.
- ViewModel is a presentation state machine.
- Model/domain owns business meaning.
- Services/repositories own external effects.
- Dependencies are injected.
- Navigation ownership is a design decision.
- Async UI is state transitions.
- Tests verify behavior, not implementation details.

## Junior Questions

What is MVVM?

Model-View-ViewModel. It separates data, UI, and presentation logic.

What does a ViewModel do?

It prepares data for the view and handles user actions.

Why use MVVM?

To make UI code cleaner and logic easier to test.

## Mid-Level Questions

What goes in a ViewModel?

Presentation state, validation, formatting, async loading, error mapping, and user action handling.

What should not go in a ViewModel?

Direct UI layout, direct networking construction, persistence implementation details, and unrelated app-wide state.

How do you test MVVM?

Inject stub services, call ViewModel actions, and assert resulting state or side effects.

## Senior Questions

How do you avoid massive ViewModels?

I scope ViewModels by feature ownership and lifetime, move external effects to services/use cases, move domain rules to domain models, split unrelated state, and avoid making one ViewModel coordinate the whole app.

How do you decide if a screen needs MVVM?

I look at behavior complexity. If the screen has async state, validation, error handling, side effects, navigation decisions, or tests, MVVM likely helps. If it is a pure display component, a ViewModel may be ceremony.

Where should navigation live?

It depends. Simple SwiftUI screens can own route state. Complex flows often use routers/coordinators. ViewModels can emit navigation intents, but they should not construct views.

How do you design error handling?

I map technical errors into display-safe errors, keep retry behavior explicit, preserve old data when refresh fails if appropriate, and avoid raw backend messages in UI.

How do you handle dependencies?

I inject protocols or concrete dependencies depending on boundary needs. External effects like networking, analytics, storage, and payment should be injected and replaceable in tests.

## Senior Artifact: MVVM Feature Blueprint

```text
Feature:
User goal:
Views:
ViewModels:
Domain models:
DTOs:
Repositories/services:
Use cases:
State enum:
Actions:
Navigation owner:
Dependencies:
Side effects:
Error mapping:
Tests:
Previews:
Tradeoffs:
```

## Senior Artifact: ViewModel State Machine

```text
State:
- idle
- loading
- loaded(data)
- empty
- failed(error)

Actions:
- appear
- refresh
- retry
- selectItem
- deleteItem

Transitions:
idle -> loading
loading -> loaded
loading -> empty
loading -> failed
failed -> loading on retry
loaded -> loaded on successful refresh
loaded -> failedBanner on failed refresh
```

## Senior Artifact: Code Review Checklist

```text
MVVM Code Review
Does the ViewModel have one clear responsibility?
Are dependencies injected?
Are async states explicit?
Are errors display-safe?
Are side effects hidden behind services?
Is navigation ownership clear?
Is state mutation main-actor safe?
Are derived values duplicated?
Can important transitions be tested?
Are previews deterministic?
Is this ViewModel necessary?
```

## Real Scenario: Checkout MVVM

Strong design:

- `CheckoutView` renders form and summary.
- `CheckoutViewModel` owns draft, validation, submitting state, display errors.
- `CheckoutUseCase` coordinates payment, order creation, analytics.
- `PaymentService` performs payment call.
- `OrderRepository` stores order result.
- `CheckoutRouter` handles success navigation.

Sketch:

```swift
@Observable
@MainActor
final class CheckoutViewModel {
    var draft = CheckoutDraft()
    var state: CheckoutState = .editing

    private let checkoutUseCase: CheckoutUseCase
    private let router: CheckoutRouter

    init(checkoutUseCase: CheckoutUseCase, router: CheckoutRouter) {
        self.checkoutUseCase = checkoutUseCase
        self.router = router
    }

    var canSubmit: Bool {
        draft.isValid && state != .submitting
    }

    func submit() async {
        guard canSubmit else { return }

        state = .submitting

        do {
            let orderID = try await checkoutUseCase.execute(draft)
            state = .completed(orderID)
            router.showConfirmation(orderID)
        } catch {
            state = .failed(DisplayError.checkoutFailed)
        }
    }
}
```

Senior discussion:

- ViewModel owns presentation.
- Use case owns business workflow.
- Router owns navigation.
- Services own side effects.
- Tests can cover validation, success, failure, and routing.

## Strong Closing Answer

MVVM is useful when it clarifies ownership. I do not use it as file-count architecture. I design ViewModels as focused, testable state machines; keep domain and side effects outside; inject dependencies; model async transitions explicitly; and make navigation ownership a conscious decision.

## Practice Prompts

1. Explain MVVM to a UIKit engineer.
2. Explain MVVM to a SwiftUI engineer.
3. Design MVVM for checkout.
4. Review a massive ViewModel and propose splits.
5. Write tests for loading success/failure.
6. Decide when MVVM is overkill.
7. Compare MVVM with Clean Architecture.
8. Explain navigation ownership options.
