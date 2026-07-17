# Day 29: TCA Feature Modeling From Requirements

## Why Feature Modeling Matters

In TCA, architecture starts before code. You translate product behavior into:

- State
- Actions
- Reducer logic
- Effects
- Dependencies
- Navigation
- Tests

A senior engineer does not begin by writing a reducer. They begin by understanding the feature's state machine.

## Example Requirement

Feature: Product search.

Requirements:

- User types a search query.
- Search waits briefly before calling API.
- Old searches cancel when query changes.
- Results show loading, empty, error, and loaded states.
- User can tap a product to open details.
- Recent searches are saved.
- Analytics tracks successful searches.

## Step 1: State

```swift
@ObservableState
struct State: Equatable {
    var query = ""
    var results: [ProductRow] = []
    var isSearching = false
    var errorMessage: String?
    var recentSearches: [String] = []
    var path = StackState<Path.State>()

    var isEmpty: Bool {
        !isSearching && errorMessage == nil && !query.isEmpty && results.isEmpty
    }
}
```

State should include what the feature needs to render and decide.

## Step 2: Actions

```swift
enum Action {
    case queryChanged(String)
    case searchResponse(Result<[Product], SearchError>)
    case productTapped(Product.ID)
    case recentSearchTapped(String)
    case clearButtonTapped
    case path(StackAction<Path.State, Path.Action>)
}
```

Actions describe events, not implementation details.

Good:

```swift
case productTapped(Product.ID)
```

Weak:

```swift
case updateNavigation
```

## Step 3: Dependencies

```swift
@Dependency(\.searchClient) var searchClient
@Dependency(\.recentSearchesClient) var recentSearchesClient
@Dependency(\.analytics) var analytics
@Dependency(\.continuousClock) var clock
```

Every outside-world interaction becomes a dependency.

## Step 4: Reducer

```swift
enum CancelID {
    case search
}

var body: some Reducer<State, Action> {
    Reduce { state, action in
        switch action {
        case .queryChanged(let query):
            state.query = query
            state.errorMessage = nil

            guard !query.isEmpty else {
                state.results = []
                state.isSearching = false
                return .cancel(id: CancelID.search)
            }

            state.isSearching = true

            return .run { send in
                try await clock.sleep(for: .milliseconds(300))
                await send(.searchResponse(
                    Result { try await searchClient.search(query) }
                ))
            }
            .cancellable(id: CancelID.search, cancelInFlight: true)

        case .searchResponse(.success(let products)):
            state.isSearching = false
            state.results = products.map(ProductRow.init)

            let query = state.query
            return .run { _ in
                await recentSearchesClient.save(query)
                await analytics.track("search_success")
            }

        case .searchResponse(.failure):
            state.isSearching = false
            state.errorMessage = "Search failed. Try again."
            return .none

        case .productTapped(let id):
            state.path.append(.details(ProductDetailsFeature.State(productID: id)))
            return .none

        case .recentSearchTapped(let query):
            state.query = query
            return .send(.queryChanged(query))

        case .clearButtonTapped:
            state.query = ""
            state.results = []
            state.errorMessage = nil
            return .cancel(id: CancelID.search)

        case .path:
            return .none
        }
    }
    .forEach(\.path, action: \.path)
}
```

## Step 5: Tests

Test the behavior the product cares about:

- Query updates state.
- Debounce waits before API.
- New query cancels old query.
- Success maps products to rows.
- Failure shows display error.
- Product tap pushes details route.

## Senior Feature Modeling Artifact

```text
Artifact: TCA Feature Modeling
Feature:
User goals:
State:
Actions:
Effects:
Dependencies:
Cancellation:
Navigation:
Error states:
Derived state:
Tests:
Known tradeoffs:
```

## Common Mistakes

- Starting with reducer code before defining behavior.
- Actions named after implementation instead of user/system events.
- State that duplicates cheap derived values.
- Dependencies hidden inside clients.
- No cancellation ID for search-like effects.
- No tests for effect responses.

## Interview Notes

Junior:

TCA features are modeled with state, actions, reducer, effects, and store.

Mid-level:

Start by listing all user/system events, then design state and effects around them.

Senior:

I model TCA features from requirements as explicit state machines. I identify outside effects, cancellation, navigation, errors, and tests before implementation.

## Practice

1. Model a login feature from requirements.
2. Model product search with debounce and cancellation.
3. Identify dependencies for checkout.
4. Write expected tests before reducer code.
