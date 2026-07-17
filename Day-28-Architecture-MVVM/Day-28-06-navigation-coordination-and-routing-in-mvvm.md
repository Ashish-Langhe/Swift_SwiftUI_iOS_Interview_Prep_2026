# Day 28: Navigation, Coordination, And Routing In MVVM

## The Navigation Question

One of the hardest MVVM decisions is:

```text
Who owns navigation?
```

Common options:

1. View owns navigation state.
2. ViewModel exposes navigation output.
3. Router/coordinator owns navigation state.
4. Parent feature owns child navigation.

There is no single correct answer. Choose based on app complexity.

## Option 1: View Owns Navigation

Good for simple SwiftUI screens.

```swift
struct ProductsScreen: View {
    @State private var viewModel: ProductsViewModel
    @State private var path: [ProductsRoute] = []

    var body: some View {
        NavigationStack(path: $path) {
            ProductsContent(
                state: viewModel.state,
                openProduct: { id in
                    path.append(.details(id))
                }
            )
            .navigationDestination(for: ProductsRoute.self) { route in
                switch route {
                case .details(let id):
                    ProductDetailsScreen(productID: id)
                }
            }
        }
    }
}
```

Pros:

- Simple.
- Navigation stays close to SwiftUI.
- ViewModel remains UI-state focused.

Cons:

- Harder to test navigation.
- Can duplicate navigation logic.

## Option 2: ViewModel Emits Navigation

```swift
@Observable
@MainActor
final class ProductsViewModel {
    var route: ProductsRoute?

    func didSelectProduct(_ id: Product.ID) {
        route = .details(id)
    }
}
```

View reacts:

```swift
.onChange(of: viewModel.route) { _, route in
    guard let route else { return }
    path.append(route)
}
```

Pros:

- Selection intent is testable.
- ViewModel can decide navigation after business checks.

Cons:

- One-shot route state can be awkward.
- ViewModel starts knowing navigation concepts.

## Option 3: Router/Coordinator

```swift
@Observable
@MainActor
final class ProductsRouter {
    var path: [ProductsRoute] = []
    var activeSheet: ProductsSheet?

    func openProduct(_ id: Product.ID) {
        path.append(.details(id))
    }

    func openFilters() {
        activeSheet = .filters
    }
}
```

View:

```swift
ProductsContent(
    state: viewModel.state,
    openProduct: router.openProduct,
    openFilters: router.openFilters
)
```

Pros:

- Centralizes navigation.
- Testable route state.
- Scales better for deep links.

Cons:

- More moving parts.
- Can become a god router if not scoped.

## Route Types

```swift
enum ProductsRoute: Hashable {
    case details(Product.ID)
    case reviews(Product.ID)
}

enum ProductsSheet: Identifiable {
    case filters
    case share(Product.ID)

    var id: String {
        switch self {
        case .filters: "filters"
        case .share(let id): "share-\(id)"
        }
    }
}
```

Use stable IDs, not large mutable models.

## UIKit Coordinator MVVM

UIKit MVVM often uses coordinators:

```swift
protocol ProductsCoordinator: AnyObject {
    func showProductDetails(id: Product.ID)
}

final class ProductsViewModel {
    weak var coordinator: ProductsCoordinator?

    func didSelectProduct(_ id: Product.ID) {
        coordinator?.showProductDetails(id: id)
    }
}
```

Use weak references to avoid cycles.

## Deep Links

For deep links, a router is usually cleaner.

```swift
func handle(_ deepLink: DeepLink) {
    switch deepLink {
    case .product(let id):
        selectedTab = .home
        homePath = [.details(id)]
    case .settings:
        selectedTab = .account
        accountPath = [.settings]
    }
}
```

## Common Mistakes

- ViewModel directly creating views.
- Passing full models as routes.
- One huge global router for everything.
- Navigation booleans scattered across many views.
- Lost deep link after login.
- UIKit coordinators retained strongly by ViewModels.

## Senior iOS Engineer Artifact

```text
Artifact: MVVM Navigation Decision
Feature:
Navigation owner:
Route types:
Sheets:
Deep links:
Auth gate:
Testing strategy:
UIKit/SwiftUI difference:
Why this approach:
Risks:
```

## Interview Notes

Junior:

Navigation moves from one screen to another.

Mid-level:

In MVVM, navigation can live in the View, ViewModel, or a router/coordinator.

Senior:

I choose navigation ownership based on complexity. Simple SwiftUI views can own paths; larger flows benefit from typed routers/coordinators. ViewModels should not construct views, and route values should be stable and testable.

## Practice

1. Implement view-owned navigation for a product list.
2. Move navigation into a router.
3. Add a sheet enum.
4. Explain how UIKit coordinator MVVM avoids retain cycles.
