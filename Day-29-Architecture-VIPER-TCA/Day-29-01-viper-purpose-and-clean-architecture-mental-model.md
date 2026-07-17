# Day 29: VIPER Purpose And Clean Architecture Mental Model

## What VIPER Is

VIPER stands for:

- View
- Interactor
- Presenter
- Entity
- Router

VIPER applies Clean Architecture ideas to iOS screens. It separates UI, business logic, presentation logic, data models, and navigation into distinct components.

Simple flow:

```text
View -> Presenter -> Interactor -> Entity/Service
View <- Presenter <- Interactor
Presenter -> Router
```

## Why VIPER Exists

VIPER was created to fight common iOS architecture problems:

- Massive View Controllers
- Business logic in views
- Hard-to-test UI code
- Navigation scattered everywhere
- Unclear responsibility boundaries
- Large screens that become risky to change

VIPER makes each module explicit and testable.

## VIPER Components

View:

- Displays UI.
- Sends user events to Presenter.
- Contains no business logic.
- Usually `UIViewController`, `UIView`, or SwiftUI wrapper.

Presenter:

- Handles user interaction logic.
- Requests work from Interactor.
- Converts results into view display models.
- Tells View what to render.
- Tells Router when to navigate.

Interactor:

- Contains use-case/business logic.
- Talks to repositories/services.
- Does not know about UI.
- Returns results to Presenter.

Entity:

- Domain model used by Interactor.
- Represents business data.
- Should not depend on UI.

Router:

- Builds modules.
- Handles navigation.
- Owns screen transitions.

## Basic VIPER Example

Feature: Product details.

Entity:

```swift
struct Product {
    let id: ProductID
    let name: String
    let price: Decimal
    let isAvailable: Bool
}
```

Interactor:

```swift
protocol ProductDetailsInteractorInput {
    func loadProduct(id: ProductID) async throws -> Product
}

final class ProductDetailsInteractor: ProductDetailsInteractorInput {
    private let repository: ProductRepository

    init(repository: ProductRepository) {
        self.repository = repository
    }

    func loadProduct(id: ProductID) async throws -> Product {
        try await repository.product(id: id)
    }
}
```

Presenter:

```swift
@MainActor
final class ProductDetailsPresenter {
    weak var view: ProductDetailsViewInput?

    private let productID: ProductID
    private let interactor: ProductDetailsInteractorInput
    private let router: ProductDetailsRouterInput

    init(
        productID: ProductID,
        interactor: ProductDetailsInteractorInput,
        router: ProductDetailsRouterInput
    ) {
        self.productID = productID
        self.interactor = interactor
        self.router = router
    }

    func viewDidLoad() {
        view?.showLoading()

        Task {
            do {
                let product = try await interactor.loadProduct(id: productID)
                view?.show(ProductDetailsDisplayModel(product: product))
            } catch {
                view?.showError("Unable to load product.")
            }
        }
    }
}
```

View:

```swift
protocol ProductDetailsViewInput: AnyObject {
    func showLoading()
    func show(_ model: ProductDetailsDisplayModel)
    func showError(_ message: String)
}
```

Router:

```swift
protocol ProductDetailsRouterInput {
    func showReviews(productID: ProductID)
}
```

## When VIPER Helps

VIPER is useful when:

- App is large.
- Teams work in parallel.
- Screens have complex business use cases.
- Strong test boundaries matter.
- Navigation is complicated.
- UIKit codebase needs clean separation.
- Modules are owned by different teams.

## When VIPER Hurts

VIPER may be too heavy when:

- App is small.
- Screen is simple.
- Team moves fast with few engineers.
- SwiftUI state-driven approach is enough.
- Boilerplate slows development more than it helps.

## Senior iOS Engineer Perspective

Senior engineers do not pick VIPER because it sounds enterprise. They pick it when module boundaries, testing, and separation outweigh boilerplate.

Ask:

- Is this feature complex enough?
- Will separate layers clarify ownership?
- Can the team maintain the boilerplate?
- Are use cases testable?
- Does routing need isolation?
- Is the screen likely to grow?

## Senior Artifact

```text
Artifact: VIPER Fit Check
Feature:
Business complexity:
Navigation complexity:
Team size:
Testing need:
UIKit or SwiftUI:
Module ownership:
Boilerplate cost:
Decision:
```

## Common Mistakes

- VIPER for every tiny screen.
- Presenter doing business logic.
- Interactor formatting UI strings.
- Router knowing business rules.
- View retaining Presenter strongly while Presenter strongly retains View.
- Too many protocols with no testing value.

## Interview Notes

Junior:

VIPER separates a screen into View, Interactor, Presenter, Entity, and Router.

Mid-level:

The Presenter coordinates UI logic, Interactor owns business logic, and Router owns navigation.

Senior:

I use VIPER when strict boundaries and testability matter enough to justify boilerplate. I keep components focused and avoid turning VIPER into ceremony without architectural value.

## Practice

1. Identify VIPER components in a login screen.
2. Decide if a simple settings row needs VIPER.
3. Move business logic from Presenter to Interactor.
4. Explain VIPER's relationship to Clean Architecture.
