# Day 30: Dependency Injection

## What Dependency Injection Means

Dependency Injection means a type receives the objects it needs instead of creating them internally.

Bad:

```swift
final class ProductsViewModel {
    private let service = LiveProductService()
}
```

Better:

```swift
final class ProductsViewModel {
    private let service: ProductService

    init(service: ProductService) {
        self.service = service
    }
}
```

The ViewModel no longer decides which service implementation exists. It only depends on the capability.

## Why DI Matters

DI improves:

- Testability
- Previewability
- Modularity
- Replaceability
- Separation of concerns
- Dependency visibility

It also prevents hidden coupling to live networking, databases, analytics, and global state.

## Protocol-Based DI

```swift
protocol ProductService {
    func products() async throws -> [Product]
}

struct LiveProductService: ProductService {
    func products() async throws -> [Product] {
        // URLSession call
    }
}

struct StubProductService: ProductService {
    let productsToReturn: [Product]

    func products() async throws -> [Product] {
        productsToReturn
    }
}
```

Use in ViewModel:

```swift
@Observable
@MainActor
final class ProductsViewModel {
    var state: LoadState<[Product]> = .idle

    private let service: ProductService

    init(service: ProductService) {
        self.service = service
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

## Initializer Injection

Initializer injection is the most explicit form.

```swift
final class CheckoutViewModel {
    private let paymentService: PaymentService
    private let analytics: AnalyticsClient

    init(paymentService: PaymentService, analytics: AnalyticsClient) {
        self.paymentService = paymentService
        self.analytics = analytics
    }
}
```

Use it for required dependencies.

## Property Injection

```swift
final class Logger {
    var analytics: AnalyticsClient?
}
```

Use carefully. Property injection can create invalid partially configured objects.

## Method Injection

```swift
func export(report: Report, using exporter: ReportExporter) throws {
    try exporter.export(report)
}
```

Use when dependency is needed only for one operation.

## Dependency Container

```swift
struct AppDependencies {
    let productService: ProductService
    let authService: AuthService
    let analytics: AnalyticsClient

    func makeProductsViewModel() -> ProductsViewModel {
        ProductsViewModel(service: productService)
    }
}
```

A dependency container centralizes construction without using global singletons everywhere.

## SwiftUI Environment DI

```swift
private struct AnalyticsKey: EnvironmentKey {
    static let defaultValue: AnalyticsClient = NoopAnalyticsClient()
}

extension EnvironmentValues {
    var analytics: AnalyticsClient {
        get { self[AnalyticsKey.self] }
        set { self[AnalyticsKey.self] = newValue }
    }
}
```

Use:

```swift
struct CheckoutView: View {
    @Environment(\.analytics) private var analytics

    var body: some View {
        Button("Pay") {
            analytics.track("pay_tapped")
        }
    }
}
```

## When To Use DI

Use DI for:

- Networking
- Persistence
- Analytics
- Feature flags
- Auth/session services
- Date/UUID generation in test-sensitive code
- Payment/location/notification clients

Avoid overdoing DI for:

- Simple value formatters
- Pure functions
- Tiny local helpers
- Dependencies with no alternate implementation or test need

## Senior iOS Engineer Perspective

Ask:

- Is this dependency visible?
- Can I test with a fake?
- Can previews run without live services?
- Is this required at initialization?
- Is this protocol useful or speculative?
- Is a container clarifying construction?

## Common Mistakes

- Creating live services inside ViewModels.
- Global singletons for everything.
- Protocols for every tiny class.
- Property injection causing invalid objects.
- Dependency containers used as service locators everywhere.
- Tests accidentally hitting real network.

## Interview Notes

Junior:

DI means passing dependencies into a type.

Mid-level:

Initializer injection is explicit and testable. Protocols allow live and fake implementations.

Senior:

I use DI to make effects explicit and replaceable. I avoid both hidden singletons and speculative protocol soup, and I choose initializer, method, environment, or container injection based on ownership and lifetime.

## Practice

1. Refactor a ViewModel that creates its own service.
2. Create a stub service for previews.
3. Build an `AppDependencies` container.
4. Explain when a protocol is unnecessary.
