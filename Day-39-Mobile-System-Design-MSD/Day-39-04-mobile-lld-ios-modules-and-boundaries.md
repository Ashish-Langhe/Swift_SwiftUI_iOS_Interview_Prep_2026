# Day 39: Mobile LLD, iOS Modules, And Boundaries

Low-level design in MSD explains how the iOS app is organized internally. A good LLD answer shows testable boundaries, state ownership, data mapping, dependency injection, concurrency rules, and where feature code lives.

## Generic iOS LLD

```text
Feature View
  -> ViewModel / Reducer
  -> Use Case / Interactor
  -> Repository
  -> API Client + Local Store
  -> Mapper / DTO / Domain Model
```

This flow keeps UI from knowing network details and keeps API DTOs from leaking across the app.

## Recommended Module Layers

### App Shell

Owns:

- App launch.
- Dependency assembly.
- Session bootstrap.
- Root navigation.
- Deep link routing.
- Global app lifecycle events.

Example:

```swift
@main
struct CommerceApp: App {
    @State private var container = AppContainer.live

    var body: some Scene {
        WindowGroup {
            RootView(
                store: RootStore(
                    sessionService: container.sessionService,
                    deepLinkRouter: container.deepLinkRouter
                )
            )
        }
    }
}
```

### Feature Module

Owns one product capability:

- Feed.
- Search.
- Cart.
- Checkout.
- Chat.
- Profile.
- Payments.

Feature module includes:

- Screen views.
- View models or reducers.
- Feature-specific state.
- Feature use cases.
- Feature routing.
- Feature tests.

### Domain Layer

Owns business meaning:

```swift
struct Order: Identifiable, Equatable {
    let id: OrderID
    let status: OrderStatus
    let items: [OrderItem]
    let total: Money
}

enum OrderStatus: Equatable {
    case placed
    case preparing
    case pickedUp(driver: DriverSummary)
    case delivered
    case cancelled(reason: String)
}
```

Domain models should be stable and app-friendly. API DTOs can change more often.

### Data Layer

Owns:

- API clients.
- DTOs.
- Local persistence.
- Cache policy.
- Sync queue.
- Mapping DTOs to domain.

Example:

```swift
protocol ProductRepository {
    func product(id: Product.ID) async throws -> Product
    func cachedProduct(id: Product.ID) async -> Product?
}

final class LiveProductRepository: ProductRepository {
    private let api: ProductAPI
    private let cache: ProductCache

    init(api: ProductAPI, cache: ProductCache) {
        self.api = api
        self.cache = cache
    }

    func product(id: Product.ID) async throws -> Product {
        let dto = try await api.fetchProduct(id: id)
        let product = ProductMapper.map(dto)
        await cache.save(product)
        return product
    }

    func cachedProduct(id: Product.ID) async -> Product? {
        await cache.product(id: id)
    }
}
```

## State Ownership

State mistakes are common in MSD interviews.

Use this rule:

```text
View owns temporary UI state.
Feature state owns screen behavior.
Repository/cache owns persisted data.
Session service owns authentication state.
Server owns globally consistent business state.
```

Example:

- Search text: view or feature state.
- Current selected tab: app/root state.
- Access token: session/auth service.
- Cart total: server source of truth, locally cached.
- Draft message: local store until sent.

## Navigation Boundary

Navigation should not be hidden inside networking code or repositories.

Possible approaches:

- SwiftUI `NavigationStack` with route enum.
- Coordinator for UIKit.
- Router service for deep links.
- TCA navigation state for reducer-driven flows.

Example:

```swift
enum AppRoute: Hashable {
    case home
    case product(id: Product.ID)
    case cart
    case orderTracking(id: Order.ID)
}
```

Senior note:

```text
Deep links should resolve through auth and data availability. A push should not blindly navigate to a screen that cannot load.
```

## Dependency Injection

Use DI to make modules testable and replaceable.

```swift
struct FeedDependencies {
    let repository: FeedRepository
    let imageLoader: ImageLoading
    let analytics: AnalyticsTracking
}

@MainActor
final class FeedViewModel: ObservableObject {
    @Published private(set) var state: FeedState = .idle

    private let dependencies: FeedDependencies

    init(dependencies: FeedDependencies) {
        self.dependencies = dependencies
    }
}
```

Avoid:

- Creating network clients directly inside views.
- Using global singletons for everything.
- Passing huge app containers into every feature.

## Concurrency Boundary

UI state should update on the main actor.

```swift
@MainActor
final class ProductViewModel: ObservableObject {
    @Published private(set) var state: ProductState = .loading
    private let repository: ProductRepository

    init(repository: ProductRepository) {
        self.repository = repository
    }

    func load(id: Product.ID) {
        Task {
            do {
                let product = try await repository.product(id: id)
                state = .loaded(product)
            } catch {
                state = .failed(error.localizedDescription)
            }
        }
    }
}
```

Senior detail:

- Keep heavy parsing off the main actor.
- Use actors for shared mutable services.
- Make DTOs and domain models `Sendable` when crossing concurrency boundaries.
- Cancel tasks when screens disappear if work is no longer useful.

## LLD Review Checklist

- Are DTOs separated from domain models?
- Can the feature be tested without real network?
- Is navigation explicit?
- Is local cache behind an interface?
- Does the view model own too much?
- Are retries centralized?
- Is state modeled as enum where appropriate?
- Are async updates main-actor safe?
- Can this feature move to a package later?

## Interview Notes

- LLD is where senior iOS depth shines.
- Do not list patterns; show ownership.
- Explain why a boundary exists.
- Keep examples small but realistic.
- Mention tests and dependency injection naturally.
