# Day 27: Advanced Navigation And Deep Linking

## Why Advanced Navigation Matters

Basic navigation is easy. Production navigation is harder because apps need:

- Tabs
- Multiple stacks
- Sheets
- Deep links
- Universal links
- Notifications
- State restoration
- Split view on iPad
- Authentication gates

SwiftUI navigation should be modeled as data.

## Route Modeling

```swift
enum AppRoute: Hashable, Codable {
    case product(Product.ID)
    case order(Order.ID)
    case settings
    case supportTicket(String)
}
```

Routes should use stable values, usually IDs. Avoid passing large mutable models.

## Router

```swift
@Observable
@MainActor
final class AppRouter {
    var selectedTab: AppTab = .home
    var homePath: [AppRoute] = []
    var accountPath: [AppRoute] = []
    var activeSheet: ActiveSheet?

    func openProduct(_ id: Product.ID) {
        selectedTab = .home
        homePath.append(.product(id))
    }

    func openSettings() {
        selectedTab = .account
        accountPath.append(.settings)
    }

    func reset() {
        homePath.removeAll()
        accountPath.removeAll()
        activeSheet = nil
    }
}
```

Each tab can preserve its own navigation stack.

## Root Composition

```swift
struct RootView: View {
    @State private var router = AppRouter()

    var body: some View {
        TabView(selection: $router.selectedTab) {
            NavigationStack(path: $router.homePath) {
                HomeView()
                    .navigationDestination(for: AppRoute.self, destination: destination)
            }
            .tabItem { Label("Home", systemImage: "house") }
            .tag(AppTab.home)

            NavigationStack(path: $router.accountPath) {
                AccountView()
                    .navigationDestination(for: AppRoute.self, destination: destination)
            }
            .tabItem { Label("Account", systemImage: "person") }
            .tag(AppTab.account)
        }
        .environment(router)
    }

    @ViewBuilder
    private func destination(for route: AppRoute) -> some View {
        switch route {
        case .product(let id):
            ProductDetailsScreen(productID: id)
        case .order(let id):
            OrderDetailsScreen(orderID: id)
        case .settings:
            SettingsScreen()
        case .supportTicket(let id):
            SupportTicketScreen(ticketID: id)
        }
    }
}
```

## Deep Link Parsing

```swift
struct DeepLinkParser {
    func route(from url: URL) -> AppRoute? {
        let components = url.pathComponents

        if components.count >= 3, components[1] == "products" {
            return .product(Product.ID(rawValue: components[2]))
        }

        if components.count >= 3, components[1] == "orders" {
            return .order(Order.ID(rawValue: components[2]))
        }

        return nil
    }
}
```

Router handles the result:

```swift
.onOpenURL { url in
    guard let route = parser.route(from: url) else { return }
    router.handle(route)
}
```

## Authentication Gates

Deep links may require login.

```swift
func handle(_ route: AppRoute, session: SessionModel) {
    if route.requiresAuthentication, !session.isLoggedIn {
        activeSheet = .loginAfterSuccess(route)
        return
    }

    navigate(to: route)
}
```

Senior note: do not drop the user's intended route after login.

## NavigationSplitView

For iPad/macOS, use split navigation.

```swift
NavigationSplitView {
    Sidebar(selection: $selectedSection)
} content: {
    ItemList(section: selectedSection, selection: $selectedItem)
} detail: {
    if let selectedItem {
        ItemDetail(itemID: selectedItem)
    } else {
        ContentUnavailableView("Select an Item", systemImage: "sidebar.left")
    }
}
```

Advanced apps adapt route state across compact and regular layouts.

## Latest SwiftUI Notes

Apple's newer navigation guidance continues to emphasize `NavigationStack` and `NavigationSplitView` over deprecated `NavigationView`. Recent updates also add more item-driven navigation destination overloads and split-view column control, reinforcing state-driven navigation.

## Common Mistakes

- Passing full models in routes.
- One global navigation path for all tabs.
- Dropping deep links behind auth.
- Mixing sheet state and stack state randomly.
- Duplicating destination switches across the app.
- Ignoring iPad split-view behavior.

## Senior iOS Engineer Perspective

Senior navigation design answers:

- What is the route model?
- Who owns each stack?
- How are deep links parsed and validated?
- How is auth handled?
- What happens on logout?
- Can routes be restored?
- Does this adapt to iPad?
- Can tests assert navigation state?

## Interview Notes

Junior:

Use `NavigationStack` to move between screens.

Mid-level:

Use route enums and bound paths for programmatic navigation.

Senior:

I model navigation as stable route state, support deep links and auth gates, keep tab stacks independent, centralize destination mapping, and adapt to `NavigationSplitView` for larger screens.

## Practice

1. Build a route enum with product and settings routes.
2. Add separate paths for two tabs.
3. Parse a product deep link.
4. Explain how you preserve an intended route through login.
