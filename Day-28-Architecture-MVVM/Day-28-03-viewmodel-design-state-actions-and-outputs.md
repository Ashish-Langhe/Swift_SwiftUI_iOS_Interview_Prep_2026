# Day 28: ViewModel Design, State, Actions, And Outputs

## ViewModel Responsibility

A ViewModel should answer:

- What is the current presentation state?
- What user actions are supported?
- What side effects are needed?
- How do results change state?
- What should the view render?

It should not be a random object where all code goes.

## State Enum

Use state enums for mutually exclusive states.

```swift
enum LoadState<Value: Equatable>: Equatable {
    case idle
    case loading
    case loaded(Value)
    case empty
    case failed(String)
}
```

Use:

```swift
@Observable
@MainActor
final class OrdersViewModel {
    var state: LoadState<[OrderRowViewModel]> = .idle

    private let repository: OrdersRepository

    init(repository: OrdersRepository) {
        self.repository = repository
    }
}
```

This is better than scattered booleans:

```swift
var isLoading = false
var orders: [Order] = []
var errorMessage: String?
var isEmpty = false
```

Booleans can conflict. Enums make illegal states harder.

## Actions

User actions should map to clear methods.

```swift
func load() async
func refresh() async
func retry() async
func selectOrder(_ id: Order.ID)
func deleteOrder(_ id: Order.ID) async
```

The view should not decide business behavior.

```swift
Button("Retry") {
    Task { await viewModel.retry() }
}
```

## Input And Output Style

Some teams prefer action enums.

```swift
enum OrdersAction {
    case appeared
    case refresh
    case retry
    case selected(Order.ID)
    case delete(Order.ID)
}

func send(_ action: OrdersAction) {
    switch action {
    case .appeared:
        Task { await loadIfNeeded() }
    case .refresh:
        Task { await refresh() }
    case .retry:
        Task { await load() }
    case .selected(let id):
        selectedOrderID = id
    case .delete(let id):
        Task { await delete(id) }
    }
}
```

This can be useful when:

- You want reducer-like structure.
- You want event logging.
- You want tests around actions.
- The feature has many interactions.

For smaller screens, direct methods are simpler.

## Row View Models

For complex formatting, create display models.

```swift
struct OrderRowViewModel: Identifiable, Equatable {
    let id: Order.ID
    let title: String
    let subtitle: String
    let amountText: String
    let statusText: String
}
```

Mapper:

```swift
extension OrderRowViewModel {
    init(order: Order, currencyCode: String) {
        id = order.id
        title = "Order #\(order.number)"
        subtitle = order.createdAt.formatted(date: .abbreviated, time: .omitted)
        amountText = order.total.formatted(.currency(code: currencyCode))
        statusText = order.status.title
    }
}
```

This keeps heavy formatting out of rows.

## Avoid ViewModel Knowing Layout

Bad:

```swift
var shouldUseTwoColumns: Bool
var cardWidth: CGFloat
```

Those are usually view/layout concerns.

Better:

```swift
var sections: [DashboardSection]
```

The View decides layout based on size class/container.

## Derived State

Prefer computed derived state when cheap.

```swift
var canSubmit: Bool {
    !email.isEmpty && password.count >= 8 && !isSubmitting
}
```

Store derived state only when:

- It is expensive.
- It comes from async work.
- It needs caching.
- It represents a meaningful transition.

## Main Actor

UI ViewModels should usually be main-actor isolated.

```swift
@Observable
@MainActor
final class LoginViewModel {
    var email = ""
    var password = ""
    var state: LoginState = .idle
}
```

This makes UI state mutation safer.

## Senior iOS Engineer Artifact

```text
Artifact: ViewModel API Review
ViewModel:
Owned state:
Derived state:
Actions:
Async effects:
Dependencies:
Navigation output:
Error mapping:
Thread/actor isolation:
Test cases:
State enum complete:
```

## Common Mistakes

- Too many booleans instead of a state enum.
- Public mutable properties for everything.
- ViewModel directly controlling layout.
- Formatting repeatedly inside SwiftUI rows.
- Async methods that leave loading state stuck.
- ViewModels created inside `body`.
- No clear action API.

## Interview Notes

Junior:

A ViewModel stores data the View shows and handles actions.

Mid-level:

Use state enums, clear action methods, injected services, and display models for formatting.

Senior:

I design ViewModels as state machines with explicit actions, actor-safe mutations, dependency boundaries, display-specific outputs, and tests for important transitions.

## Practice

1. Replace `isLoading`, `items`, and `error` with one state enum.
2. Add row view models for formatted order rows.
3. Convert button logic into ViewModel actions.
4. Explain why layout should usually stay out of the ViewModel.
