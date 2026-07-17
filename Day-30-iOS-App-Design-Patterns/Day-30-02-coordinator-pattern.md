# Day 30: Coordinator Pattern

## What Coordinator Pattern Means

The Coordinator pattern moves navigation flow out of view controllers or views and into dedicated coordinator objects.

It is common in UIKit and still useful in SwiftUI for larger flows.

Without coordinator:

```swift
final class ProductsViewController: UIViewController {
    func didSelectProduct(_ product: Product) {
        let details = ProductDetailsViewController(product: product)
        navigationController?.pushViewController(details, animated: true)
    }
}
```

With coordinator:

```swift
protocol ProductsCoordinator: AnyObject {
    func showProductDetails(id: Product.ID)
}

final class ProductsViewController: UIViewController {
    weak var coordinator: ProductsCoordinator?

    func didSelectProduct(_ product: Product) {
        coordinator?.showProductDetails(id: product.id)
    }
}
```

## Why Coordinators Matter

Coordinators improve:

- Navigation separation
- Flow reuse
- Deep-link handling
- UIKit module construction
- Testability of route decisions
- Retain-cycle control

## Basic UIKit Coordinator

```swift
final class AppCoordinator {
    private let navigationController: UINavigationController
    private let dependencies: AppDependencies

    init(
        navigationController: UINavigationController,
        dependencies: AppDependencies
    ) {
        self.navigationController = navigationController
        self.dependencies = dependencies
    }

    func start() {
        showProducts()
    }

    func showProducts() {
        let viewModel = ProductsViewModel(service: dependencies.productService)
        let viewController = ProductsViewController(viewModel: viewModel)
        viewController.coordinator = self
        navigationController.setViewControllers([viewController], animated: false)
    }
}

extension AppCoordinator: ProductsCoordinator {
    func showProductDetails(id: Product.ID) {
        let details = ProductDetailsAssembly.build(
            productID: id,
            dependencies: dependencies
        )
        navigationController.pushViewController(details, animated: true)
    }
}
```

## Child Coordinators

Use child coordinators for flows.

```swift
final class CheckoutCoordinator {
    private let navigationController: UINavigationController
    private let dependencies: AppDependencies
    var onFinish: (() -> Void)?

    func start(cart: Cart) {
        showShipping(cart: cart)
    }

    private func showShipping(cart: Cart) {
        // build shipping screen
    }
}
```

Parent keeps child alive:

```swift
private var childCoordinators: [AnyObject] = []
```

Remove child on finish to avoid leaks.

## SwiftUI Coordinator / Router

SwiftUI often models coordinator as route state.

```swift
@Observable
@MainActor
final class AppRouter {
    var path: [AppRoute] = []
    var activeSheet: ActiveSheet?

    func openProduct(_ id: Product.ID) {
        path.append(.productDetails(id))
    }

    func openCheckout(cartID: Cart.ID) {
        path.append(.checkout(cartID))
    }
}
```

Use:

```swift
NavigationStack(path: $router.path) {
    ProductsView()
        .navigationDestination(for: AppRoute.self) { route in
            destination(for: route)
        }
}
```

## Coordinator vs Router

Coordinator:

- Often UIKit object.
- Owns navigation controller.
- Builds screens.
- Manages child flows.

Router:

- Often SwiftUI state object.
- Owns routes/path/sheets.
- Maps routes to destinations.
- Works well with value-driven navigation.

## When To Use Coordinator

Use when:

- Navigation is complex.
- UIKit codebase has many pushes/modals.
- Multiple screens form one flow.
- Deep links need centralized handling.
- ViewModels should not construct views.

Skip when:

- Simple SwiftUI screen.
- Navigation state is local and obvious.
- Adding coordinator only creates ceremony.

## Common Mistakes

- View controller still builds all next screens.
- Coordinator strongly retained by child and child strongly retained by coordinator.
- One giant coordinator for whole app.
- Child coordinator never removed.
- Business logic in coordinator.
- SwiftUI coordinator duplicates navigation path state.

## Senior Artifact

```text
Artifact: Coordinator Flow Review
Flow:
Coordinator owner:
Navigation container:
Child coordinators:
Start route:
Finish route:
Deep links:
Memory ownership:
Dependencies:
Testing:
```

## Interview Notes

Junior:

Coordinator separates navigation from view controllers.

Mid-level:

Coordinators build screens and manage flows; child coordinators handle subflows.

Senior:

I use coordinators when navigation complexity needs ownership. I manage child lifetime carefully, avoid business logic in coordinators, and use route-state routers for SwiftUI where that fits better.

## Practice

1. Move navigation from a view controller to a coordinator.
2. Create a checkout child coordinator.
3. Design a SwiftUI `AppRouter`.
4. Explain coordinator memory ownership.
