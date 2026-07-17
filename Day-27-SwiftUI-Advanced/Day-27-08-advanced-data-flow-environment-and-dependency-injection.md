# Day 27: Advanced Data Flow, Environment, And Dependency Injection

## Why This Topic Matters

Large SwiftUI apps fail when data flow becomes invisible.

Advanced apps need:

- Feature-scoped models
- App/session state
- Dependency injection
- Environment values
- Testable services
- Clear ownership
- Controlled mutation

## Ownership Layers

Common layers:

- View-owned state: `@State`
- Editable parent state: `@Binding`
- Feature model: `@Observable`
- App/session model: environment
- Service/repository: dependency
- Persistent state: storage layer

Each layer should have a reason.

## Feature Model

```swift
@Observable
@MainActor
final class CartModel {
    var items: [CartItem] = []
    var isCheckingOut = false
    var errorMessage: String?

    @ObservationIgnored
    private let checkoutService: CheckoutService

    init(checkoutService: CheckoutService) {
        self.checkoutService = checkoutService
    }

    func checkout() async {
        isCheckingOut = true
        defer { isCheckingOut = false }

        do {
            try await checkoutService.checkout(items)
        } catch {
            errorMessage = "Checkout failed."
        }
    }
}
```

Dependencies are ignored by observation because they are not UI state.

## Environment Dependency

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

## App Container

A simple dependency container can be useful.

```swift
struct AppDependencies {
    let analytics: AnalyticsClient
    let productService: ProductService
    let checkoutService: CheckoutService
}
```

Inject at the root:

```swift
RootView()
    .environment(\.analytics, dependencies.analytics)
```

Or pass services into feature model factories.

## Avoid Global Singletons

Weak:

```swift
Analytics.shared.track("checkout")
```

Better:

```swift
@Environment(\.analytics) private var analytics
```

The second version is easier to test and preview.

## Environment vs Initializer Injection

Use initializer injection for required feature-specific dependencies.

```swift
ProductsScreen(model: ProductsModel(service: dependencies.productService))
```

Use environment for ambient dependencies used broadly:

- Analytics
- Theme
- Session
- Locale/config
- Feature flags

Senior rule: if a view cannot make sense without a dependency, initializer injection is often clearer.

## Mutation Boundaries

Do not let every view mutate shared state arbitrarily.

```swift
@Observable
final class SessionModel {
    private(set) var user: User?

    func signIn(user: User) {
        self.user = user
    }

    func signOut() {
        user = nil
    }
}
```

Expose commands, not random mutable internals.

## Common Mistakes

- Putting everything in environment.
- Single global app model with unrelated state.
- Services observed as UI state.
- Views constructing live dependencies.
- Hidden mutation from many leaf views.
- Previews requiring real backend setup.

## Senior iOS Engineer Perspective

Senior data-flow design asks:

- Who owns this state?
- Who can mutate it?
- Is this state or dependency?
- Is environment hiding too much?
- Can previews override this?
- Can tests inject fakes?
- Is mutation command-based?

## Interview Notes

Junior:

SwiftUI passes data through state, bindings, and environment.

Mid-level:

Use observable models for feature state and environment for shared dependencies.

Senior:

I design explicit ownership layers, inject dependencies, limit shared mutable state, keep services out of observation, and make previews/tests cheap.

## Practice

1. Create a custom environment value for analytics.
2. Build a feature model with injected service.
3. Replace a singleton with environment injection.
4. Explain initializer injection vs environment.
