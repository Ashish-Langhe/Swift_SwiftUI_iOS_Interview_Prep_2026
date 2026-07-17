# Day 25: Navigation

## SwiftUI Navigation Mental Model

Navigation is state.

Instead of imperatively pushing and popping view controllers, SwiftUI navigation is usually modeled as values:

- Selected item
- Navigation path
- Route enum
- Presented sheet
- Tab selection

## `NavigationStack`

Basic example:

```swift
struct ProductsScreen: View {
    let products: [Product]

    var body: some View {
        NavigationStack {
            List(products) { product in
                NavigationLink(product.name, value: product.id)
            }
            .navigationTitle("Products")
            .navigationDestination(for: Product.ID.self) { productID in
                ProductDetailsScreen(productID: productID)
            }
        }
    }
}
```

`NavigationLink(value:)` pushes a value. `navigationDestination` maps that value to a destination.

## Route Enums

For real apps, route enums often scale better.

```swift
enum AppRoute: Hashable {
    case productDetails(Product.ID)
    case orderDetails(Order.ID)
    case settings
}

struct RootView: View {
    @State private var path: [AppRoute] = []

    var body: some View {
        NavigationStack(path: $path) {
            HomeView(
                openProduct: { path.append(.productDetails($0)) },
                openSettings: { path.append(.settings) }
            )
            .navigationDestination(for: AppRoute.self) { route in
                switch route {
                case .productDetails(let id):
                    ProductDetailsScreen(productID: id)
                case .orderDetails(let id):
                    OrderDetailsScreen(orderID: id)
                case .settings:
                    SettingsScreen()
                }
            }
        }
    }
}
```

This gives type-safe navigation.

## Programmatic Navigation

```swift
Button("Open Settings") {
    path.append(.settings)
}
```

Pop:

```swift
path.removeLast()
```

Pop to root:

```swift
path.removeAll()
```

## Deep Linking

Navigation paths make deep linking easier.

```swift
func handle(url: URL) {
    guard let productID = Product.ID(url.lastPathComponent) else {
        return
    }

    path = [.productDetails(productID)]
}
```

For multi-step routes:

```swift
path = [
    .productDetails(productID),
    .reviews(productID)
]
```

## Sheets and Full-Screen Covers

Use sheet state for modal presentation.

```swift
enum ActiveSheet: Identifiable {
    case filters
    case share(Product.ID)

    var id: String {
        switch self {
        case .filters:
            return "filters"
        case .share(let id):
            return "share-\(id)"
        }
    }
}

struct ProductsScreen: View {
    @State private var activeSheet: ActiveSheet?

    var body: some View {
        ProductList(
            openFilters: { activeSheet = .filters },
            shareProduct: { activeSheet = .share($0) }
        )
        .sheet(item: $activeSheet) { sheet in
            switch sheet {
            case .filters:
                FiltersView()
            case .share(let id):
                ShareProductView(productID: id)
            }
        }
    }
}
```

This avoids many separate boolean sheet flags.

## Navigation and Identity

Navigation values should be stable and hashable.

Good:

```swift
case productDetails(Product.ID)
```

Risky:

```swift
case productDetails(Product)
```

Passing a large mutable model as a route can create stale data or hashing issues. Prefer IDs and let the destination load or observe the source of truth.

## Navigation and Architecture

For small apps, local navigation state is fine.

For larger apps, create a coordinator-style model:

```swift
@Observable
final class AppRouter {
    var path: [AppRoute] = []
    var activeSheet: ActiveSheet?

    func openProduct(_ id: Product.ID) {
        path.append(.productDetails(id))
    }

    func popToRoot() {
        path.removeAll()
    }
}
```

Use it:

```swift
struct RootView: View {
    @State private var router = AppRouter()

    var body: some View {
        NavigationStack(path: $router.path) {
            HomeView()
                .environment(router)
                .navigationDestination(for: AppRoute.self) { route in
                    destination(for: route)
                }
        }
    }

    @ViewBuilder
    private func destination(for route: AppRoute) -> some View {
        switch route {
        case .productDetails(let id):
            ProductDetailsScreen(productID: id)
        case .settings:
            SettingsScreen()
        }
    }
}
```

This centralizes navigation behavior without forcing every view to know the full navigation system.

## Latest SwiftUI Navigation Adjacent Notes

Recent SwiftUI releases have focused on richer toolbar behavior, presentation APIs, and container interactions. Navigation remains closely tied to these areas because real apps combine stacks, sheets, toolbars, swipe actions, and adaptive layouts.

Senior takeaway: navigation should be value-driven, testable, and stable across app lifecycle events.

## Common Mistakes

- Using many booleans for mutually exclusive sheets.
- Passing full mutable models through route values.
- Creating destinations inline in ways that duplicate business logic.
- Mixing navigation ownership across many unrelated views.
- Losing deep-link support because navigation is too imperative.

## Senior iOS Engineer Perspective

Senior navigation design asks:

- What owns the navigation state?
- Are routes type-safe?
- Are route values stable and hashable?
- Can deep links reconstruct the path?
- Can tests verify navigation intents?
- Does the destination load fresh data or depend on stale passed state?
- Does this scale to iPad, split view, tabs, and modals?

## Interview Notes

Junior:

Use `NavigationStack`, `NavigationLink`, and `navigationDestination` to move between SwiftUI screens.

Mid-level:

Navigation is state. You can bind a path and push route values programmatically.

Senior:

I model navigation as stable route values, often with a route enum and path. I keep ownership clear, avoid passing large mutable models, support deep links, and test navigation through intent callbacks or router state.

## Practice

1. Build a `NavigationStack` with product details.
2. Replace multiple sheet booleans with one `ActiveSheet` enum.
3. Add a route enum and programmatic push.
4. Explain why route values should usually be IDs.
