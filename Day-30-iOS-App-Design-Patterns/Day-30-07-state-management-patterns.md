# Day 30: State Management Patterns

## Why State Management Matters

Most iOS bugs are state bugs:

- Loading and error conflict.
- Form resets unexpectedly.
- Navigation loses route.
- Search result race.
- UI displays stale data.
- Shared state updates from wrong place.

State management means deciding:

- Who owns state?
- Who can mutate it?
- How does UI observe it?
- How are transitions tested?

## Local View State

Use local state for UI-only values.

```swift
@State private var isShowingFilters = false
@State private var searchText = ""
```

Good for:

- Sheet visibility
- Toggle state
- Search text
- Temporary form draft

## Observable Feature State

Use observable models for feature state.

```swift
@Observable
@MainActor
final class ProductsModel {
    var state: LoadState<[Product]> = .idle

    func load() async {
        // update state
    }
}
```

Good for:

- Async loading
- Validation
- Error handling
- Feature workflow

## State Enum

```swift
enum LoadState<Value> {
    case idle
    case loading
    case loaded(Value)
    case empty
    case failed(String)
}
```

State enums prevent impossible combinations.

Bad:

```swift
var isLoading = true
var errorMessage = "Failed"
var products: [Product] = []
```

Is this loading or failed? Enum makes it clear.

## App-Level State

Use app-level state for:

- Session
- Selected tab
- Feature flags
- Global routing
- User profile

Keep it scoped.

Bad:

```swift
final class AppState {
    var user: User?
    var cart: Cart
    var products: [Product]
    var settings: Settings
    var checkoutDraft: CheckoutDraft
}
```

Better:

- `SessionModel`
- `CartModel`
- `AppRouter`
- Feature-specific models

## Reducer State

TCA/reducer style:

```swift
struct State: Equatable {
    var count = 0
}

enum Action {
    case incrementTapped
}

func reduce(state: inout State, action: Action) {
    switch action {
    case .incrementTapped:
        state.count += 1
    }
}
```

Reducer style is strong when transitions and effects need exhaustive testing.

## Shared State

Shared state needs one owner.

Example cart:

- Cart model owns items.
- Product details sends "add item" intent.
- Checkout reads cart.

Avoid many features mutating shared arrays independently.

## Derived State

Do not store cheap derived state.

```swift
var canSubmit: Bool {
    email.contains("@") && password.count >= 8 && !isSubmitting
}
```

Store derived state only when:

- Expensive to compute.
- Needs caching.
- Represents async result.

## Common Mistakes

- Too many booleans.
- No single owner.
- Global state for feature-local data.
- Storing derived state that drifts.
- Mutating UI state off main actor.
- Views doing business state transitions.
- Shared mutable state without tests.

## Senior Artifact

```text
Artifact: State Ownership Review
Feature:
Local state:
Feature state:
App state:
Shared state:
Owner:
Mutators:
Derived state:
Async transitions:
Tests:
```

## Interview Notes

Junior:

State is data that changes and updates UI.

Mid-level:

Use local state for UI details, observable models for feature state, and enums for loading/error states.

Senior:

I design state around ownership, mutation boundaries, impossible-state prevention, actor isolation, and testable transitions. I avoid global mutable state unless the scope is truly global.

## Practice

1. Replace three booleans with a state enum.
2. Identify owner of cart state.
3. Move derived state into a computed property.
4. Compare observable model vs reducer state.
