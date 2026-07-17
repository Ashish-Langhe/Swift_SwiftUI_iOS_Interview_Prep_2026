# Day 25: Observable State

## Why Observable State Exists

Local `@State` is enough for simple view-owned values. Real apps need richer state:

- Async loading
- Error handling
- Caching
- Form validation
- Navigation coordination
- Business rules
- Dependency calls

Observable state lets a model object change and have SwiftUI update the views that read it.

## Modern Observation with `@Observable`

```swift
import Observation
import SwiftUI

@Observable
final class ProductsModel {
    var products: [Product] = []
    var isLoading = false
    var errorMessage: String?

    private let service: ProductService

    init(service: ProductService) {
        self.service = service
    }

    @MainActor
    func load() async {
        isLoading = true
        errorMessage = nil

        do {
            products = try await service.fetchProducts()
        } catch {
            errorMessage = "Unable to load products."
        }

        isLoading = false
    }
}
```

Use it in SwiftUI:

```swift
struct ProductsScreen: View {
    @State private var model = ProductsModel(service: LiveProductService())

    var body: some View {
        List(model.products) { product in
            Text(product.name)
        }
        .overlay {
            if model.isLoading {
                ProgressView()
            }
        }
        .task {
            await model.load()
        }
    }
}
```

## Observation vs `ObservableObject`

Older SwiftUI commonly used:

```swift
final class ProductsViewModel: ObservableObject {
    @Published var products: [Product] = []
}
```

Modern SwiftUI can use:

```swift
@Observable
final class ProductsModel {
    var products: [Product] = []
}
```

The newer Observation system tracks property access more precisely.

## Choosing State Ownership

Use `@State` to own an observable model inside a view:

```swift
@State private var model = ProductsModel(service: LiveProductService())
```

Pass it to child views:

```swift
ProductList(model: model)
```

Create bindings inside child views with `@Bindable` if needed:

```swift
struct ProfileForm: View {
    @Bindable var model: ProfileModel

    var body: some View {
        TextField("Name", text: $model.name)
    }
}
```

## Real iOS Example: View State Enum

```swift
enum Loadable<Value> {
    case idle
    case loading
    case loaded(Value)
    case failed(String)
}

@Observable
final class OrdersModel {
    var state: Loadable<[Order]> = .idle
    private let service: OrdersService

    init(service: OrdersService) {
        self.service = service
    }

    @MainActor
    func load() async {
        state = .loading

        do {
            state = .loaded(try await service.orders())
        } catch {
            state = .failed("Could not load orders.")
        }
    }
}
```

Render:

```swift
struct OrdersScreen: View {
    @State private var model: OrdersModel

    var body: some View {
        Group {
            switch model.state {
            case .idle, .loading:
                ProgressView()
            case .loaded(let orders):
                OrdersList(orders: orders)
            case .failed(let message):
                ContentUnavailableView("Error", systemImage: "exclamationmark.triangle", description: Text(message))
            }
        }
        .task {
            await model.load()
        }
    }
}
```

This is better than scattered booleans like `isLoading`, `hasError`, and `orders`.

## Avoid Massive View Models

A common mistake is creating one giant observable object:

```swift
@Observable
final class AppViewModel {
    var user: User?
    var products: [Product] = []
    var cart: Cart = .empty
    var settings = Settings()
    var navigationPath = NavigationPath()
}
```

This becomes hard to test and reason about.

Prefer feature-scoped models:

- `SessionModel`
- `ProductsModel`
- `CartModel`
- `SettingsModel`

## Observable State and Dependencies

Inject dependencies instead of creating them deep inside methods.

```swift
@Observable
final class LoginModel {
    var email = ""
    var password = ""
    var errorMessage: String?

    private let authService: AuthService

    init(authService: AuthService) {
        self.authService = authService
    }

    @MainActor
    func signIn() async -> Bool {
        do {
            try await authService.signIn(email: email, password: password)
            return true
        } catch {
            errorMessage = "Invalid email or password."
            return false
        }
    }
}
```

This makes testing and previewing easier.

## Senior iOS Engineer Perspective

Senior observable-state design is about boundaries:

- Keep UI rendering in views.
- Keep state transitions in models.
- Keep networking in services.
- Make async behavior testable.
- Use enums for mutually exclusive states.
- Avoid global observable objects unless the state is truly global.
- Make ownership and lifetime obvious.

## Interview Notes

Junior:

Observable state updates SwiftUI when model data changes.

Mid-level:

Modern SwiftUI uses Observation with `@Observable` and `@Bindable`; older apps often use `ObservableObject` with `@Published`.

Senior:

I design observable models around feature ownership, explicit state transitions, dependency injection, and main-actor UI safety. I avoid god objects, duplicated state, and views that mix rendering with business rules.

## Practice

1. Build a `LoginModel` with email, password, loading, and error state.
2. Refactor multiple booleans into a `Loadable` enum.
3. Inject a mock service into an observable model.
4. Explain when you would keep state in a view versus a model.
