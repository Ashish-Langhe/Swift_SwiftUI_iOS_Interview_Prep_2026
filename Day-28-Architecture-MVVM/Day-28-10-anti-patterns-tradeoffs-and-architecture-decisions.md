# Day 28: Anti-Patterns, Tradeoffs, And Architecture Decisions

## MVVM Anti-Patterns

MVVM fails when boundaries are unclear.

This file is the senior-level warning label.

## Anti-Pattern: Massive ViewModel

```swift
final class DashboardViewModel {
    var user: User?
    var products: [Product] = []
    var cart: Cart = .empty
    var settings: Settings = .default
    var path: [Route] = []
    var analytics: AnalyticsClient
    var authService: AuthService
    var productService: ProductService
}
```

Problem:

- Too many responsibilities.
- Hard to test.
- High update frequency.
- Hard to reuse.
- Changes become risky.

Better:

- `SessionViewModel`
- `ProductsViewModel`
- `CartViewModel`
- `SettingsViewModel`
- Feature router

## Anti-Pattern: ViewModel As Service Locator

Bad:

```swift
viewModel.apiClient.paymentService.analytics.track(...)
```

ViewModels should not expose dependency graphs.

## Anti-Pattern: Business Logic In View

Bad:

```swift
Button("Pay") {
    if cart.total > 0 && user.isVerified && network.isReachable {
        Task { try await paymentService.pay(cart) }
    }
}
```

Better:

```swift
Button("Pay") {
    Task { await viewModel.pay() }
}
.disabled(!viewModel.canPay)
```

## Anti-Pattern: ViewModel Creates Views

Bad:

```swift
func detailsView(for product: Product) -> some View {
    ProductDetailsView(product: product)
}
```

ViewModels should not construct SwiftUI views. Use routers/coordinators or destination builders in the view layer.

## Anti-Pattern: Everything Gets A ViewModel

Bad:

```swift
ProductTitleViewModel
ProductPriceViewModel
ProductSubtitleViewModel
```

For simple pure views, pass values directly.

## Anti-Pattern: No Explicit State

Bad:

```swift
var products: [Product] = []
var isLoading = false
var error: Error?
var didLoad = false
```

Better:

```swift
enum State {
    case idle
    case loading
    case loaded([Product])
    case failed(DisplayError)
}
```

## Tradeoff: ViewModel Per Screen vs Per Feature

Per screen:

- Easier local reasoning.
- Smaller objects.
- Can duplicate shared logic.

Per feature:

- Shared state across screens.
- Easier coordination.
- Can grow too large.

Senior answer: scope by ownership and lifetime.

## Tradeoff: Protocol Everything?

Protocols help testing and modularity.

But too many protocols create noise.

Use protocols when:

- Multiple implementations exist.
- You need test doubles.
- Boundary crosses modules.
- Dependency has external effects.

Skip protocols when:

- Type is pure value logic.
- No alternate implementation needed.
- The abstraction is speculative.

## Tradeoff: MVVM vs MVC vs VIPER vs TCA

MVC:

- Simple.
- Can become massive controllers.

MVVM:

- Good balance for many iOS apps.
- Testable presentation logic.
- Can produce massive ViewModels.

VIPER/Clean:

- Strong boundaries.
- More files and ceremony.

TCA/reducer architecture:

- Very explicit state/actions/effects.
- Strong testability.
- Learning curve and framework commitment.

Senior answer: choose based on team, complexity, test needs, and long-term maintenance.

## Architecture Decision Record

```text
ADR: Use MVVM for Product Search
Context:
Product search has debounce, pagination, error recovery, filters, and deep link selection.

Decision:
Use a SearchViewModel with injected SearchRepository and SearchRouter.

Alternatives:
- Logic in View
- Reducer architecture
- Shared global app model

Consequences:
- State transitions are testable.
- More files than a simple view.
- Router must stay feature-scoped.
```

## Common Mistakes

- Choosing architecture by trend.
- No written decision for complex features.
- Refactoring architecture mid-feature without tests.
- Overengineering small screens.
- Underengineering critical flows.
- Confusing folder structure with architecture.

## Senior iOS Engineer Artifact

```text
Artifact: MVVM Architecture Decision
Feature:
Complexity:
Team familiarity:
Test requirements:
Alternative patterns:
Chosen pattern:
Why:
Tradeoffs:
Exit criteria:
Review date:
```

## Interview Notes

Junior:

MVVM helps separate UI and logic.

Mid-level:

MVVM has tradeoffs. It improves testability but can create too much ceremony.

Senior:

I evaluate architecture by ownership, complexity, testability, team fit, and maintenance cost. I document meaningful decisions and watch for massive ViewModels, hidden dependencies, and speculative abstractions.

## Practice

1. Identify anti-patterns in a massive ViewModel.
2. Write an ADR for using MVVM in search.
3. Decide whether a simple row needs a ViewModel.
4. Compare MVVM and TCA for a checkout flow.
