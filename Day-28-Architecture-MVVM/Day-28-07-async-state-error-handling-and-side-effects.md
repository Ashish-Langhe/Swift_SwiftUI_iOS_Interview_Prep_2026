# Day 28: Async State, Error Handling, And Side Effects

## Why Async State Is Central To MVVM

Most real ViewModels coordinate asynchronous work:

- Network requests
- Database reads
- Authentication
- Uploads
- Search
- Pagination
- Refresh
- Retry

Async work creates state transitions. MVVM should make those transitions explicit.

## Loading State

```swift
enum ScreenState<Value: Equatable>: Equatable {
    case idle
    case loading
    case loaded(Value)
    case empty
    case failed(DisplayError)
}

struct DisplayError: Equatable {
    let title: String
    let message: String
    let retryTitle: String
}
```

Use display-safe errors. Do not show raw backend messages directly.

## Basic Async Load

```swift
@Observable
@MainActor
final class ProductsViewModel {
    var state: ScreenState<[ProductRowViewModel]> = .idle

    private let repository: ProductRepository

    init(repository: ProductRepository) {
        self.repository = repository
    }

    func load() async {
        state = .loading

        do {
            let products = try await repository.products()
            state = products.isEmpty
                ? .empty
                : .loaded(products.map(ProductRowViewModel.init))
        } catch {
            state = .failed(DisplayError(
                title: "Unable to Load Products",
                message: "Check your connection and try again.",
                retryTitle: "Retry"
            ))
        }
    }
}
```

## Refresh

Refresh may differ from initial loading.

```swift
func refresh() async {
    do {
        let products = try await repository.products()
        state = products.isEmpty
            ? .empty
            : .loaded(products.map(ProductRowViewModel.init))
    } catch {
        bannerMessage = "Refresh failed."
    }
}
```

Senior note: during refresh, you may want to keep old data visible and show a non-blocking error.

## Retry

```swift
func retry() async {
    await load()
}
```

For write operations, retry is more complex. Retrying a payment, order, or transfer can duplicate side effects unless the backend supports idempotency.

## Cancellation

```swift
@ObservationIgnored
private var searchTask: Task<Void, Never>?

func searchTextChanged(_ text: String) {
    query = text
    searchTask?.cancel()

    searchTask = Task { [weak self] in
        try? await Task.sleep(for: .milliseconds(300))
        guard !Task.isCancelled else { return }
        await self?.search()
    }
}
```

Cancel stale work. Prevent old responses from overwriting new state.

## Side Effects

Side effects include:

- Network calls
- Disk writes
- Analytics
- Haptics
- Navigation
- Notifications
- Keychain access

Keep side effects behind injected dependencies.

```swift
protocol AnalyticsClient {
    func track(_ event: String)
}
```

## Optimistic Update

```swift
func toggleFavorite(productID: Product.ID) async {
    guard let index = products.firstIndex(where: { $0.id == productID }) else {
        return
    }

    products[index].isFavorite.toggle()
    let newValue = products[index].isFavorite

    do {
        try await repository.setFavorite(newValue, productID: productID)
    } catch {
        products[index].isFavorite.toggle()
        bannerMessage = "Could not update favorite."
    }
}
```

Optimistic updates need rollback.

## Actor Isolation

UI ViewModels should generally be `@MainActor`.

Heavy work should move out:

```swift
let rows = await mapper.rows(from: products)
state = .loaded(rows)
```

Do not block the main actor with expensive CPU work.

## Common Mistakes

- Loading state stuck after error.
- Retry duplicates writes.
- Raw technical error messages.
- Stale search responses.
- Side effects hidden inside views.
- No cancellation.
- Heavy work on the main actor.

## Senior iOS Engineer Artifact

```text
Artifact: Async State Review
Feature:
Initial load:
Refresh:
Retry:
Cancellation:
Error mapping:
Optimistic updates:
Idempotency risk:
Side effects:
Actor isolation:
Tests:
```

## Interview Notes

Junior:

Async ViewModels load data and update state.

Mid-level:

Handle loading, success, empty, error, retry, and refresh states.

Senior:

I design async ViewModels as state machines with cancellation, stale response protection, display-safe error mapping, injected side effects, idempotency awareness, and main-actor UI mutation.

## Practice

1. Add loading/empty/error states to a ViewModel.
2. Add non-blocking refresh behavior.
3. Add cancellation to search.
4. Explain why retrying writes can be dangerous.
