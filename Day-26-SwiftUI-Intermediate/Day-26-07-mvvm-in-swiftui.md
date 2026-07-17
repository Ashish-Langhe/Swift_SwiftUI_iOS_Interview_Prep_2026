# Day 26: MVVM in SwiftUI

## What MVVM Means

MVVM means:

- Model: domain/data types and services.
- View: SwiftUI rendering and user interactions.
- ViewModel: presentation state and actions.

In SwiftUI, the term "ViewModel" needs care because SwiftUI already has state-driven rendering. A ViewModel should not become a dumping ground.

## Basic MVVM Example

```swift
struct Product: Identifiable, Equatable {
    let id: UUID
    let name: String
    let price: Decimal
}

protocol ProductService {
    func products() async throws -> [Product]
}

@Observable
@MainActor
final class ProductsViewModel {
    enum State: Equatable {
        case idle
        case loading
        case loaded([Product])
        case failed(String)
    }

    var state: State = .idle
    var query = ""

    private let service: ProductService

    init(service: ProductService) {
        self.service = service
    }

    var visibleProducts: [Product] {
        guard case .loaded(let products) = state else {
            return []
        }

        return products.filter {
            query.isEmpty ||
            $0.name.localizedCaseInsensitiveContains(query)
        }
    }

    func load() async {
        state = .loading

        do {
            state = .loaded(try await service.products())
        } catch {
            state = .failed("Unable to load products.")
        }
    }
}
```

View:

```swift
struct ProductsScreen: View {
    @State private var viewModel: ProductsViewModel

    var body: some View {
        content
            .searchable(text: $viewModel.query)
            .task {
                await viewModel.load()
            }
    }

    @ViewBuilder
    private var content: some View {
        switch viewModel.state {
        case .idle, .loading:
            ProgressView()
        case .loaded:
            List(viewModel.visibleProducts) { product in
                ProductRow(product: product)
            }
        case .failed(let message):
            ContentUnavailableView("Error", systemImage: "exclamationmark.triangle", description: Text(message))
        }
    }
}
```

## MVVM Is Not Mandatory Everywhere

Small views do not need ViewModels.

```swift
struct PriceLabel: View {
    let price: Decimal

    var body: some View {
        Text(price.formatted(.currency(code: "USD")))
    }
}
```

Adding a ViewModel here would be ceremony.

## What Belongs in a ViewModel

Good ViewModel responsibilities:

- Feature state
- Loading and error transitions
- Presentation-friendly derived values
- User intents
- Dependency coordination
- Validation
- Navigation intents if the architecture chooses that

Poor ViewModel responsibilities:

- Direct view layout decisions
- Creating real network clients internally
- Holding app-wide unrelated state
- Formatting every tiny label if it makes the model huge
- Storing duplicated derived state

## Dependency Injection

```swift
struct StubProductService: ProductService {
    func products() async throws -> [Product] {
        [.sample]
    }
}

#Preview {
    ProductsScreen(
        viewModel: ProductsViewModel(service: StubProductService())
    )
}
```

ViewModels should be easy to create with test services.

## Testing ViewModels

```swift
@Test
func load_success_setsLoadedState() async {
    let viewModel = ProductsViewModel(service: StubProductService())

    await viewModel.load()

    #expect(viewModel.visibleProducts.count == 1)
}
```

Test transitions, not private implementation details.

## MVVM with Navigation

Two common choices:

1. View owns navigation state and ViewModel exposes user actions/data.
2. Router owns navigation state and ViewModel sends navigation intents.

Example intent:

```swift
enum ProductsAction {
    case openDetails(Product.ID)
}

@Observable
final class ProductsRouter {
    var path: [ProductsRoute] = []

    func handle(_ action: ProductsAction) {
        switch action {
        case .openDetails(let id):
            path.append(.details(id))
        }
    }
}
```

Senior tip: avoid making every ViewModel know every route in the app.

## ViewModel Lifetime

If a screen owns a ViewModel:

```swift
@State private var viewModel = ProductsViewModel(service: service)
```

If an ancestor owns it, pass it down.

```swift
ProductListView(viewModel: viewModel)
```

Do not recreate ViewModels accidentally inside `body`.

Bad:

```swift
var body: some View {
    ProductsListView(viewModel: ProductsViewModel(service: service))
}
```

That can reset state repeatedly.

## MVVM and Observation

Modern SwiftUI often uses `@Observable` ViewModels instead of `ObservableObject` + `@Published`.

Older:

```swift
final class ProductsViewModel: ObservableObject {
    @Published var products: [Product] = []
}
```

Modern:

```swift
@Observable
final class ProductsViewModel {
    var products: [Product] = []
}
```

Use `@Bindable` when a view needs bindings into observable properties.

## Common Mistakes

- Creating ViewModels in `body`.
- Adding ViewModels for tiny pure views.
- Giant ViewModels for entire app sections.
- ViewModels directly constructing live services.
- Mixing navigation, networking, storage, validation, and formatting without boundaries.
- ViewModels exposing too much mutable state.

## Senior iOS Engineer Perspective

Senior MVVM in SwiftUI is pragmatic:

- Use ViewModels when state transitions are non-trivial.
- Keep views declarative.
- Keep services injected.
- Keep feature state focused.
- Test async transitions.
- Avoid ceremony for simple views.
- Respect SwiftUI's native state model.

## Interview Notes

Junior:

MVVM separates UI from state/business logic.

Mid-level:

In SwiftUI, ViewModels often hold loading state, errors, dependencies, and user actions.

Senior:

I use MVVM selectively. SwiftUI already gives state-driven rendering, so I introduce ViewModels where they clarify ownership, async flows, validation, and testability. I avoid god ViewModels and accidental lifecycle bugs.

## Practice

1. Build a `ProductsViewModel` with loading/error/loaded state.
2. Inject a stub service.
3. Write a test for successful loading.
4. Remove an unnecessary ViewModel from a simple row view.
