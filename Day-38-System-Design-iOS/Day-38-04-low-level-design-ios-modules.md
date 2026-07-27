# Day 38: Low-Level Design For iOS Modules

LLD is where a system design answer becomes practical iOS engineering. It defines modules, protocols, models, state, ownership, dependencies, and testability.

## LLD Goal

LLD should answer:

- What classes/structs/protocols exist?
- Who owns state?
- Who performs network calls?
- Who maps data?
- Who handles navigation?
- Who stores data?
- How is the module tested?

## Example Feature: Product List

Recommended module shape:

```text
ProductListView
ProductListViewController / SwiftUI View
ProductListViewModel
ProductListState
ProductListAction
ProductRepository
ProductRemoteDataSource
ProductLocalDataSource
ProductDTO
Product
ProductRow
ProductListCoordinator
```

## Model Layers

### DTO

Network response shape.

```swift
struct ProductDTO: Decodable {
    let id: String
    let title: String?
    let imageURL: URL?
    let price: PriceDTO?
}
```

### Domain Model

App meaning.

```swift
struct Product: Identifiable, Equatable {
    let id: String
    let title: String
    let imageURL: URL?
    let price: Decimal
    let currencyCode: String
}
```

### Row Model

UI display shape.

```swift
struct ProductRow: Identifiable, Hashable {
    let id: String
    let titleText: String
    let priceText: String
    let imageURL: URL?
}
```

## View State

```swift
enum ProductListState: Equatable {
    case idle
    case loading
    case loaded(ProductListContent)
    case empty
    case failed(message: String)
}

struct ProductListContent: Equatable {
    let rows: [ProductRow]
    let isRefreshing: Bool
    let isLoadingNextPage: Bool
    let paginationError: String?
    let canLoadMore: Bool
}
```

Why:

- First-page failure is different from next-page failure.
- Refresh should not necessarily remove old content.
- UI becomes a function of state.

## Repository Protocol

```swift
protocol ProductRepository {
    func cachedProducts() async throws -> [Product]
    func fetchProducts(cursor: String?) async throws -> ProductPage
}

struct ProductPage {
    let products: [Product]
    let nextCursor: String?
}
```

## ViewModel

```swift
@MainActor
final class ProductListViewModel {
    private let repository: ProductRepository
    private let mapper: ProductRowMapping
    private var nextCursor: String?
    private var isLoadingNextPage = false

    private(set) var state: ProductListState = .idle

    init(repository: ProductRepository, mapper: ProductRowMapping) {
        self.repository = repository
        self.mapper = mapper
    }

    func load() async {
        state = .loading

        do {
            let page = try await repository.fetchProducts(cursor: nil)
            nextCursor = page.nextCursor
            let rows = page.products.map(mapper.map)
            state = rows.isEmpty ? .empty : .loaded(
                ProductListContent(
                    rows: rows,
                    isRefreshing: false,
                    isLoadingNextPage: false,
                    paginationError: nil,
                    canLoadMore: nextCursor != nil
                )
            )
        } catch {
            state = .failed(message: "Unable to load products.")
        }
    }
}
```

## Mapper

```swift
protocol ProductRowMapping {
    func map(_ product: Product) -> ProductRow
}

struct ProductRowMapper: ProductRowMapping {
    let priceFormatter: PriceFormatting

    func map(_ product: Product) -> ProductRow {
        ProductRow(
            id: product.id,
            titleText: product.title,
            priceText: priceFormatter.string(
                from: product.price,
                currencyCode: product.currencyCode
            ),
            imageURL: product.imageURL
        )
    }
}
```

## Navigation

Navigation should be an output, not hidden inside the cell.

```swift
protocol ProductListCoordinating: AnyObject {
    func showProductDetails(id: String)
}

func didSelectProduct(id: String) {
    coordinator.showProductDetails(id: id)
}
```

## Dependency Injection

```swift
enum ProductListAssembly {
    static func make(
        apiClient: APIClient,
        localStore: ProductLocalStore,
        coordinator: ProductListCoordinating
    ) -> ProductListViewController {
        let repository = LiveProductRepository(
            apiClient: apiClient,
            localStore: localStore
        )
        let mapper = ProductRowMapper(priceFormatter: CurrencyPriceFormatter())
        let viewModel = ProductListViewModel(repository: repository, mapper: mapper)
        return ProductListViewController(viewModel: viewModel, coordinator: coordinator)
    }
}
```

## Testing Plan

Test:

- DTO mapping.
- Empty response.
- Network failure.
- Cache fallback.
- Pagination.
- Refresh.
- Price formatting.
- Selection navigation.
- Duplicate IDs.
- Stale request cancellation.

## LLD Interview Checklist

- Clear models.
- Clear state enum.
- Repository abstraction.
- Mapper abstraction.
- Injected dependencies.
- Stable identity.
- Main actor UI state.
- Error mapping.
- Test cases.

## Common Mistakes

- View controller does networking.
- UI uses DTOs directly.
- All state is booleans.
- Cells navigate.
- No stable row IDs.
- No fake repository for tests.
- No pagination state.
- No cancellation strategy.

