# Day 25: SwiftUI Fundamentals Interview Guide

## One-Minute Senior Answer

SwiftUI is a declarative UI framework where views are value descriptions derived from state. I design SwiftUI screens by identifying state ownership, passing bindings only across editing boundaries, using observable models for feature state, using environment for ambient dependencies, keeping identity stable, building adaptive layouts, and modeling navigation as values. At senior level, SwiftUI skill is less about memorizing modifiers and more about understanding data flow, identity, lifetime, accessibility, performance, and architecture.

## Core Mental Models

- `View`: a lightweight value that describes UI.
- `body`: a function of state.
- `@State`: view-owned mutable state.
- `@Binding`: child access to parent-owned state.
- `@Observable`: model state that views can observe.
- `@Bindable`: bindings into observable models.
- `@Environment`: ambient subtree-scoped values and dependencies.
- Identity: determines state lifetime and diffing behavior.
- Layout: parent proposes, child chooses, parent places.
- Navigation: route state, not just screen pushing.

## Junior-Level Questions

What is SwiftUI?

SwiftUI is Apple's declarative framework for building UI using Swift.

What is a view?

A view is a value that describes part of the UI.

What is `@State`?

It stores local mutable state owned by a view.

What is `@Binding`?

It lets a child view read and write state owned by a parent.

What is `NavigationStack`?

It is SwiftUI's stack-based navigation container.

## Mid-Level Questions

Why are SwiftUI views structs?

They are lightweight value descriptions. SwiftUI can recreate them often and manage persistent storage separately through identity and property wrappers.

When do you use `@State` vs `@Binding`?

Use `@State` when the view owns the value. Use `@Binding` when the parent owns the value and the child edits it.

When do you use observable state?

Use observable state when a feature needs richer mutable state, async loading, validation, or shared behavior across multiple views.

What is view identity?

Identity tells SwiftUI whether a view or item is the same logical element across updates. It affects `@State`, animations, list diffing, navigation, and tasks.

## Senior-Level Questions

How do you design state ownership in a SwiftUI feature?

I classify state by lifetime and owner. Local UI details stay in `@State`; parent-owned editable values flow through bindings; feature state lives in observable models; app/session services live in environment; persisted data belongs in storage or domain services. I avoid duplicating derived state and make async transitions explicit.

How do you prevent SwiftUI views from becoming unmaintainable?

I keep rendering declarative, move business transitions into models or services, extract subviews around domain meaning, inject dependencies, write previews for important states, and test state transitions. I avoid giant views, giant global models, and hidden environment coupling.

How do you debug random state resets?

I inspect identity. I check conditional branches, `ForEach` IDs, computed UUIDs, explicit `.id`, route values, and whether a parent recreated a subtree. Most random SwiftUI state resets are identity or ownership problems.

How do you design navigation at scale?

I model navigation as route values. For simple screens, local path state is fine. For larger flows, I use a router or coordinator model with typed routes, stable IDs, sheet enums, and deep-link reconstruction. I avoid passing large mutable models as routes.

## Senior iOS Engineer Artifact

```text
Artifact: SwiftUI Feature Design Review
Feature:
Screen owner:
Local state:
Parent-owned state:
Observable feature model:
Environment dependencies:
Navigation routes:
Sheet/modal model:
List/item identity:
Async loading strategy:
Error/empty/loading states:
Accessibility checks:
Dynamic type checks:
Localization risks:
Preview states:
Test strategy:
Performance risks:
```

## Real Feature Walkthrough

Feature: Product list with filters and details.

State design:

```swift
@Observable
final class ProductsModel {
    var state: Loadable<[Product]> = .idle
    var filters = ProductFilters()

    private let service: ProductService

    init(service: ProductService) {
        self.service = service
    }

    var visibleProducts: [Product] {
        guard case .loaded(let products) = state else {
            return []
        }

        return products.filter { product in
            filters.query.isEmpty ||
            product.name.localizedCaseInsensitiveContains(filters.query)
        }
    }

    @MainActor
    func load() async {
        state = .loading

        do {
            state = .loaded(try await service.products())
        } catch {
            state = .failed("Could not load products.")
        }
    }
}
```

Navigation:

```swift
enum ProductsRoute: Hashable {
    case details(Product.ID)
}
```

Screen:

```swift
struct ProductsScreen: View {
    @State private var model: ProductsModel
    @State private var path: [ProductsRoute] = []
    @State private var activeSheet: ActiveSheet?

    var body: some View {
        NavigationStack(path: $path) {
            content
                .navigationTitle("Products")
                .toolbar {
                    Button("Filters") {
                        activeSheet = .filters
                    }
                }
                .navigationDestination(for: ProductsRoute.self) { route in
                    switch route {
                    case .details(let id):
                        ProductDetailsScreen(productID: id)
                    }
                }
        }
        .sheet(item: $activeSheet) { sheet in
            switch sheet {
            case .filters:
                FiltersView(filters: $model.filters)
            }
        }
        .task {
            await model.load()
        }
    }

    @ViewBuilder
    private var content: some View {
        switch model.state {
        case .idle, .loading:
            ProgressView()
        case .loaded:
            ProductList(products: model.visibleProducts) { productID in
                path.append(.details(productID))
            }
        case .failed(let message):
            ContentUnavailableView("Error", systemImage: "exclamationmark.triangle", description: Text(message))
        }
    }
}
```

Senior discussion:

- The model owns feature state.
- The screen owns navigation state.
- The filter sheet edits via binding.
- Details navigation uses stable ID.
- UI states are mutually exclusive through `Loadable`.
- Async loading is centralized.
- The list can be previewed independently.

## Latest SwiftUI Points to Mention

Recent SwiftUI updates emphasize:

- Observation-based state with `@Observable` and `@Bindable`.
- Improved state initialization behavior in newer toolchains.
- Macro-based/custom environment improvements.
- Broader container interactions such as reordering and swipe actions.
- Toolbar and presentation API improvements.
- Continued focus on performance, data flow, and declarative architecture.

In interviews, connect latest APIs back to fundamentals: state ownership, identity, layout, and testable navigation.

## Common Interview Traps

- Saying SwiftUI views are long-lived objects.
- Putting network calls directly in `body`.
- Using `@State` for parent-owned or global data.
- Passing bindings everywhere.
- Treating environment as a global variable bag.
- Using unstable IDs in lists.
- Ignoring dynamic type and localization.
- Passing full mutable models as navigation routes.
- Using `AnyView` by default.

## Strong Senior Closing Answer

At senior level, I think of SwiftUI as a state and identity system that renders UI. My job is to make ownership clear, keep routes and list IDs stable, separate rendering from business behavior, design layout for real content, and make every important state previewable and testable. The syntax is simple; the engineering skill is in making the data flow and lifecycle predictable.

## Practice Prompts

1. Design a profile edit screen and identify every state owner.
2. Build a route enum for a shopping flow.
3. Refactor a view with five booleans into state enums.
4. Fix a `ForEach` using index identity.
5. Add environment-based analytics to a feature.
6. Explain how dynamic type can break a layout.
7. Design previews for loading, empty, error, and loaded states.
8. Explain SwiftUI to a UIKit-heavy senior engineer.
