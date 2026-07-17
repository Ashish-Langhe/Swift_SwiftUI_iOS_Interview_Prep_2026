# Day 29: VIPER Module Design, Protocols, And Assembly

## What A VIPER Module Is

A VIPER module usually represents one feature or screen.

Example:

```text
ProductDetails/
  ProductDetailsViewController.swift
  ProductDetailsPresenter.swift
  ProductDetailsInteractor.swift
  ProductDetailsRouter.swift
  ProductDetailsAssembly.swift
  ProductDetailsModels.swift
```

The module owns its local flow and dependencies.

## Protocol-Oriented Boundaries

VIPER often defines protocols between components.

```swift
protocol ProductDetailsViewInput: AnyObject {
    func render(_ state: ProductDetailsViewState)
}

protocol ProductDetailsPresenterInput {
    func viewDidLoad()
    func didTapReviews()
    func didTapRetry()
}

protocol ProductDetailsInteractorInput {
    func product(id: ProductID) async throws -> Product
}

protocol ProductDetailsRouterInput {
    func showReviews(productID: ProductID)
}
```

Protocols reduce concrete coupling and make tests easier.

## View State

Instead of many view methods, a module can render a state.

```swift
enum ProductDetailsViewState: Equatable {
    case loading
    case loaded(ProductDetailsDisplayModel)
    case failed(String)
}
```

View:

```swift
func render(_ state: ProductDetailsViewState) {
    switch state {
    case .loading:
        showSpinner()
    case .loaded(let model):
        configure(with: model)
    case .failed(let message):
        showError(message)
    }
}
```

This is closer to modern state-driven UI while preserving VIPER boundaries.

## Assembly

Assembly creates and wires the module.

```swift
enum ProductDetailsAssembly {
    static func build(
        productID: ProductID,
        repository: ProductRepository
    ) -> UIViewController {
        let view = ProductDetailsViewController()
        let interactor = ProductDetailsInteractor(repository: repository)
        let router = ProductDetailsRouter(viewController: view)
        let presenter = ProductDetailsPresenter(
            productID: productID,
            interactor: interactor,
            router: router
        )

        view.presenter = presenter
        presenter.view = view

        return view
    }
}
```

Assembly keeps construction out of components.

## Router Construction

Router can build next modules.

```swift
final class ProductDetailsRouter: ProductDetailsRouterInput {
    private weak var viewController: UIViewController?
    private let repository: ProductRepository

    init(viewController: UIViewController, repository: ProductRepository) {
        self.viewController = viewController
        self.repository = repository
    }

    func showReviews(productID: ProductID) {
        let reviews = ReviewsAssembly.build(
            productID: productID,
            repository: repository
        )
        viewController?.navigationController?.pushViewController(reviews, animated: true)
    }
}
```

Router owns navigation mechanics.

## Memory Ownership

Common UIKit VIPER ownership:

```text
ViewController -> Presenter
Presenter -> Interactor
Presenter -> Router
Presenter --weak--> View
Router --weak--> ViewController
```

Use weak references where cycles can occur.

## Protocols: When To Use Them

Use protocols when:

- Component is mocked in tests.
- Boundary crosses modules.
- Multiple implementations exist.
- You want compile-time responsibility separation.

Skip protocols when:

- Module is tiny.
- No tests or alternate implementation use the abstraction.
- Protocol only repeats one concrete method and adds noise.

Senior nuance: VIPER commonly uses protocols, but protocol count should be intentional.

## SwiftUI VIPER Assembly

VIPER can be adapted to SwiftUI, but it may feel less natural.

```swift
enum ProductDetailsAssembly {
    static func build(productID: ProductID, repository: ProductRepository) -> some View {
        let interactor = ProductDetailsInteractor(repository: repository)
        let presenter = ProductDetailsPresenter(
            productID: productID,
            interactor: interactor
        )

        return ProductDetailsView(presenter: presenter)
    }
}
```

SwiftUI often prefers state-driven MVVM/TCA-style approaches. VIPER is more common in UIKit-heavy apps.

## Senior Artifact

```text
Artifact: VIPER Module Blueprint
Module:
View:
Presenter:
Interactor:
Entity models:
Router:
Assembly:
Input protocols:
Output protocols:
Dependencies:
Memory ownership:
Tests:
```

## Common Mistakes

- View creates Interactor directly.
- Router contains business logic.
- Assembly spreads across app.
- Strong reference cycle between Presenter and View.
- Protocols for everything with no testing value.
- Presenter exposes UIKit types.

## Interview Notes

Junior:

A VIPER module wires View, Presenter, Interactor, Entity, and Router together.

Mid-level:

Assembly builds the module, protocols define boundaries, and weak references avoid memory cycles.

Senior:

I design VIPER modules around ownership and construction boundaries. I use protocols where they help testing or module separation and keep assembly responsible for wiring, not behavior.

## Practice

1. Create protocols for a product details VIPER module.
2. Write an assembly function.
3. Draw ownership arrows and identify weak references.
4. Decide which protocols are useful and which are ceremony.
