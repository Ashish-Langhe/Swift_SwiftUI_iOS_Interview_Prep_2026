# Day 36.17: UIKit Architecture For Large Apps

Large UIKit apps fail when every screen becomes a massive view controller. Senior UIKit engineering is about boundaries: views render, controllers coordinate lifecycle, view models prepare state, coordinators navigate, services perform work, and models represent domain concepts.

## The Massive View Controller Problem

Symptoms:

- 1,000+ line controllers.
- API parsing inside `viewDidLoad`.
- Navigation inside cells.
- Business rules inside button actions.
- Hard-to-test screen behavior.
- Duplicate formatting logic.
- Retain cycles hidden in callbacks.

UIKit does not force this. Poor boundaries do.

## Clean UIKit Screen Structure

```text
ProductListViewController
ProductListView
ProductListViewModel
ProductListCoordinator
ProductService
ProductRepository
ProductRowMapper
```

Each object has a job:

- View: layout and rendering.
- View controller: lifecycle and binding.
- View model: state and user intent handling.
- Coordinator: navigation.
- Service/repository: data access.
- Mapper: domain to display model.

## Custom Root View

```swift
final class ProductListView: UIView {
    let collectionView: UICollectionView
    let loadingView = UIActivityIndicatorView(style: .large)
    let errorView = ErrorStateView()

    override init(frame: CGRect) {
        collectionView = UICollectionView(frame: .zero, collectionViewLayout: Self.makeLayout())
        super.init(frame: frame)
        setup()
    }

    required init?(coder: NSCoder) {
        fatalError("init(coder:) has not been implemented")
    }

    func render(_ state: ProductListViewState) {
        loadingView.isHidden = !state.isLoading
        errorView.isHidden = state.errorMessage == nil
    }
}
```

The controller can now focus on binding:

```swift
override func loadView() {
    view = productView
}
```

## View Model With State

```swift
@MainActor
final class ProductListViewModel {
    var onStateChange: ((ProductListState) -> Void)?

    private let repository: ProductRepository

    init(repository: ProductRepository) {
        self.repository = repository
    }

    func load() {
        onStateChange?(.loading)

        Task {
            do {
                let products = try await repository.fetchProducts()
                let rows = products.map(ProductRow.init)
                onStateChange?(rows.isEmpty ? .empty : .loaded(rows))
            } catch {
                onStateChange?(.failed("Unable to load products."))
            }
        }
    }
}
```

Senior note: for bigger apps, manage task cancellation explicitly.

## Coordinator

```swift
protocol ProductListCoordinating: AnyObject {
    func showProduct(id: Product.ID)
    func showFilters()
}

final class ProductListCoordinator: ProductListCoordinating {
    private weak var navigationController: UINavigationController?

    func showProduct(id: Product.ID) {
        let controller = ProductDetailsAssembly.make(productID: id)
        navigationController?.pushViewController(controller, animated: true)
    }
}
```

Why weak navigation? Coordinator ownership differs by app. Some parent coordinators own child coordinators strongly, while UIKit owns view controllers. Design ownership intentionally.

## Assembly / Factory

```swift
enum ProductListAssembly {
    static func make(coordinator: ProductListCoordinating) -> UIViewController {
        let repository = LiveProductRepository(apiClient: APIClient.shared)
        let viewModel = ProductListViewModel(repository: repository)
        return ProductListViewController(viewModel: viewModel, coordinator: coordinator)
    }
}
```

This keeps dependency construction out of the view controller.

## MVC Done Properly

UIKit's MVC is not automatically bad.

Good MVC:

- View controller handles lifecycle and UI events.
- Models are not UIKit-specific.
- Formatting is extracted when repeated.
- Networking is not inside the controller.
- Navigation is not inside cells.

Bad MVC:

- Controller owns every responsibility.

Senior answer: MVC can be acceptable for simple screens if boundaries stay disciplined.

## MVVM With UIKit

UIKit MVVM works well when:

- State is explicit.
- Inputs are methods.
- Outputs are callbacks, Combine publishers, async streams, or observable state.

Example input/output:

```swift
viewModel.onStateChange = { [weak self] state in
    self?.render(state)
}

searchView.onTextChanged = { [weak viewModel] text in
    viewModel?.search(text)
}
```

Avoid making the view model know about UIKit types like `UIViewController`, `UIColor`, or `UIImage` unless there is a deliberate boundary decision.

## Dependency Injection

Constructor injection:

```swift
init(viewModel: ProductListViewModel, coordinator: ProductListCoordinating) {
    self.viewModel = viewModel
    self.coordinator = coordinator
    super.init(nibName: nil, bundle: nil)
}
```

Benefits:

- Testable.
- Explicit dependencies.
- No hidden global state.

Use property injection only when UIKit lifecycle/storyboards require it.

## Testing Large UIKit Features

Testable pieces:

- View model transitions.
- Mapper output.
- Validation.
- Router/deep-link parsing.
- Coordinator destination decisions with fake navigation.
- Snapshot builder.

Fake repository:

```swift
struct FakeProductRepository: ProductRepository {
    let result: Result<[Product], Error>

    func fetchProducts() async throws -> [Product] {
        try result.get()
    }
}
```

## Scaling Team Ownership

Senior engineers think beyond a single file:

- Can another team add a section safely?
- Can design update cell style without touching networking?
- Can QA identify UI elements?
- Can the feature be migrated to SwiftUI later?
- Can route handling be tested?
- Can the screen survive slow network and empty states?

## Common Mistakes

- Treating MVVM as "move random code to another object."
- Creating view models that are still UIKit controllers in disguise.
- Overengineering tiny screens.
- Using global singletons for everything.
- Coordinator retain cycles.
- No state model.
- No testable mapping layer.

## Junior-Level Interview Answer

For UIKit architecture, I try to keep view controllers small. Views handle layout, view controllers handle lifecycle, view models handle state, and services handle networking or persistence.

## Senior-Level Interview Answer

I design UIKit features around ownership and change boundaries. A screen has explicit state, display models, dependency injection, navigation outputs, and testable mapping. I choose MVC for small static screens, MVVM for stateful features, coordinator for navigation-heavy flows, and avoid abstractions that do not reduce real complexity.

## Points To Remember

- Massive view controllers are a boundary failure.
- Custom root views reduce controller layout noise.
- Coordinators own navigation decisions.
- View models should not become UIKit wrappers.
- DI makes UIKit screens testable.
- Architecture should match feature complexity.

