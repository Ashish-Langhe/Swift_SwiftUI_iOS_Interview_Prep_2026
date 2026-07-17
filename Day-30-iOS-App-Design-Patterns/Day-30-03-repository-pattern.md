# Day 30: Repository Pattern

## What Repository Pattern Means

A Repository hides data source details behind a domain-friendly interface.

The ViewModel should not care whether data comes from:

- Network
- Cache
- Database
- File system
- Keychain
- Mock data

It asks the repository.

```swift
protocol ProductRepository {
    func products() async throws -> [Product]
    func product(id: Product.ID) async throws -> Product
    func saveFavorite(_ isFavorite: Bool, productID: Product.ID) async throws
}
```

## Why Repository Matters

Repository improves:

- Testability
- Offline support
- Cache strategy
- Domain boundaries
- API migration safety
- Separation from transport DTOs

## Repository With API And Cache

```swift
final class LiveProductRepository: ProductRepository {
    private let api: ProductAPI
    private let cache: ProductCache

    init(api: ProductAPI, cache: ProductCache) {
        self.api = api
        self.cache = cache
    }

    func products() async throws -> [Product] {
        if let cached = await cache.products(), !cached.isEmpty {
            return cached
        }

        let responses = try await api.products()
        let products = try responses.map(Product.init(response:))
        await cache.save(products)
        return products
    }

    func product(id: Product.ID) async throws -> Product {
        if let cached = await cache.product(id: id) {
            return cached
        }

        let response = try await api.product(id: id.rawValue)
        let product = try Product(response: response)
        await cache.save(product)
        return product
    }

    func saveFavorite(_ isFavorite: Bool, productID: Product.ID) async throws {
        try await api.saveFavorite(isFavorite, productID: productID.rawValue)
        await cache.updateFavorite(isFavorite, productID: productID)
    }
}
```

## ViewModel Uses Repository

```swift
@Observable
@MainActor
final class ProductsViewModel {
    var state: LoadState<[Product]> = .idle

    private let repository: ProductRepository

    init(repository: ProductRepository) {
        self.repository = repository
    }

    func load() async {
        state = .loading

        do {
            state = .loaded(try await repository.products())
        } catch {
            state = .failed("Unable to load products.")
        }
    }
}
```

The ViewModel does not know about API/cache details.

## Repository vs Service

Service:

- Often performs one external operation.
- Example: `ProductAPI`, `AnalyticsClient`, `PaymentService`.

Repository:

- Represents a data collection/domain boundary.
- Often combines API, cache, persistence, mapping.
- Example: `ProductRepository`, `UserRepository`.

## DTO Mapping

```swift
struct ProductResponse: Decodable {
    let id: String
    let title: String
    let price: Decimal
}

extension Product {
    init(response: ProductResponse) throws {
        guard let id = Product.ID(rawValue: response.id) else {
            throw MappingError.invalidID
        }

        self.id = id
        self.name = response.title
        self.price = response.price
    }
}
```

Repository is a natural place to map transport data into domain data.

## When To Use Repository

Use when:

- Data has multiple sources.
- Caching/offline matters.
- DTO/domain mapping matters.
- ViewModels should not know API details.
- Testing needs simple stubs.

Skip when:

- App is tiny.
- One service call directly matches the feature.
- Repository would only forward one method without adding value.

## Common Mistakes

- Repository becomes a giant god data layer.
- ViewModel still calls API directly.
- Repository returns DTOs to views.
- Cache rules hidden and untested.
- Repository performs UI formatting.
- Too many repositories with no domain boundary.

## Senior Artifact

```text
Artifact: Repository Design Review
Domain:
Repository:
Data sources:
DTO mapping:
Cache strategy:
Offline behavior:
Write behavior:
Error mapping:
Tests:
```

## Interview Notes

Junior:

Repository hides where data comes from.

Mid-level:

Repositories combine API/cache/persistence and return domain models to ViewModels.

Senior:

I use repositories at domain data boundaries where mapping, caching, offline behavior, or source orchestration matters. I avoid repositories that only add pass-through ceremony.

## Practice

1. Create a repository for products.
2. Add cache-first behavior.
3. Map DTOs into domain models.
4. Explain repository vs service.
