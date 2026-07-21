# Day 33: Dependency Inversion Principle

## What DIP Means

Dependency Inversion Principle says:

```text
High-level modules should not depend on low-level modules.
Both should depend on abstractions.
```

In iOS:

- ViewModels should not depend directly on `URLSession`.
- Use cases should not depend directly on Keychain C APIs.
- Views should not depend directly on analytics SDKs.
- Domain logic should not depend on Firebase, Core Data, or UIKit.

## Bad Example: ViewModel Depends On URLSession

```swift
@MainActor
final class ProductsViewModel {
    var products: [Product] = []

    func load() async {
        let url = URL(string: "https://api.example.com/products")!
        let data = try! await URLSession.shared.data(from: url).0
        products = try! JSONDecoder().decode([Product].self, from: data)
    }
}
```

Problems:

- Hard to test.
- ViewModel knows networking details.
- No cache/offline substitution.
- Error mapping is tangled.

## Better: Depend On Abstraction

```swift
protocol ProductRepository {
    func products() async throws -> [Product]
}

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

Low-level implementation:

```swift
struct NetworkProductRepository: ProductRepository {
    let client: HTTPClient

    func products() async throws -> [Product] {
        let responses: [ProductResponse] = try await client.get("/products")
        return try responses.map(Product.init(response:))
    }
}
```

High-level policy depends on repository abstraction.

## DIP vs Dependency Injection

Dependency Inversion is the principle.

Dependency Injection is a technique.

DIP:

```text
Depend on abstraction.
```

DI:

```text
Pass the dependency in.
```

They often work together.

## Abstraction Ownership

Senior question:

```text
Who owns the protocol?
```

Prefer the high-level consumer or domain boundary to define what it needs.

Example:

```swift
protocol CheckoutPaymentProcessing {
    func charge(_ request: CheckoutPaymentRequest) async throws -> PaymentResult
}
```

This is better than leaking a vendor SDK protocol into checkout.

## DIP With Keychain

Bad:

```swift
final class SessionManager {
    func token() throws -> String {
        // SecItemCopyMatching directly here
    }
}
```

Better:

```swift
protocol TokenStoring {
    func save(_ token: String) throws
    func token() throws -> String
    func clear() throws
}
```

Session depends on `TokenStoring`. Keychain implementation is replaceable.

## DIP With Analytics SDK

```swift
protocol AnalyticsTracking {
    func track(_ event: AnalyticsEvent)
}

struct FirebaseAnalyticsAdapter: AnalyticsTracking {
    func track(_ event: AnalyticsEvent) {
        // Firebase call
    }
}
```

Features depend on `AnalyticsTracking`, not Firebase.

## When To Use DIP

Use for:

- Network clients
- Repositories
- Persistence
- Analytics
- Payment SDKs
- Location
- Notifications
- Date/UUID/Clock in tests
- Any external effect

Avoid unnecessary abstraction for:

- Pure local value logic
- Simple models
- Stable internal helpers
- Code with no test/replace need

## Common Mistakes

- Creating protocols after concrete implementation with identical huge surface.
- Protocols owned by low-level infrastructure.
- Too many abstractions for pure functions.
- Dependency container used as global service locator.
- Feature still imports vendor SDK types.
- Tests use live implementations despite protocols.

## Senior iOS Engineer Artifact

```text
Artifact: DIP Dependency Review
High-level module:
Low-level dependency:
Current coupling:
Abstraction needed:
Protocol owner:
Injection point:
Test double:
Vendor leakage:
Tradeoff:
```

## Interview Notes

Junior:

DIP means depend on abstractions instead of concrete classes.

Mid-level:

Use protocols and injection so ViewModels can use fake services in tests.

Senior:

I apply DIP at effect and module boundaries. I keep protocols consumer-focused, avoid vendor leakage, and use dependency injection to make high-level policy independent from low-level infrastructure.

## Practice

1. Refactor URLSession out of a ViewModel.
2. Wrap Keychain behind `TokenStoring`.
3. Hide analytics SDK behind protocol.
4. Explain DIP vs DI.
