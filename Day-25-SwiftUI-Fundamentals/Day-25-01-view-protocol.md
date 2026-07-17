# Day 25: View Protocol

## What `View` Means

In SwiftUI, a screen is not built by mutating UI objects directly. A screen is described as a tree of values that conform to the `View` protocol.

```swift
import SwiftUI

struct ProfileHeaderView: View {
    let name: String
    let role: String

    var body: some View {
        VStack(alignment: .leading, spacing: 8) {
            Text(name)
                .font(.title2.bold())

            Text(role)
                .foregroundStyle(.secondary)
        }
    }
}
```

The important mental model:

- `ProfileHeaderView` is a value.
- `body` describes UI.
- SwiftUI decides how to render, diff, update, and reuse the backing platform views.
- You should describe state, identity, layout, and behavior, not manually command every UI update.

## The Protocol

Conceptually:

```swift
protocol View {
    associatedtype Body: View
    @ViewBuilder var body: Self.Body { get }
}
```

You normally do not write this conformance manually beyond `struct MyView: View`.

`Body` is an associated type, which means each view has a concrete body type. That is why SwiftUI can be strongly typed and performant.

## Why `body` Uses `some View`

Most SwiftUI view bodies return `some View`:

```swift
var body: some View {
    Text("Hello")
}
```

`some View` is an opaque return type. It hides the exact concrete type from callers while preserving a single concrete type for the compiler.

This gives you:

- Strong static type checking
- Better optimization than broad type erasure
- A cleaner public API

## `ViewBuilder` and Multiple Children

SwiftUI lets you write multiple child views inside containers because `body` uses a result builder.

```swift
var body: some View {
    VStack {
        Text("Orders")
        Text("Ready for pickup")
        Button("Refresh") {
            refresh()
        }
    }
}
```

The builder transforms that block into a concrete nested view type.

## Views Are Cheap Values

A beginner often imagines SwiftUI views as long-lived objects. They are better understood as lightweight value descriptions.

```swift
struct CounterLabel: View {
    let count: Int

    var body: some View {
        Text("Count: \(count)")
    }
}
```

When `count` changes in a parent, SwiftUI can recreate `CounterLabel`. That is normal. Do not put side effects in view initializers expecting them to run once.

## Common Mistake: Doing Work in `body`

Avoid expensive or side-effectful work in `body`.

Bad:

```swift
var body: some View {
    let products = database.fetchProducts()

    return List(products) { product in
        Text(product.name)
    }
}
```

Better:

```swift
struct ProductsView: View {
    @State private var products: [Product] = []
    let service: ProductService

    var body: some View {
        List(products) { product in
            Text(product.name)
        }
        .task {
            products = await service.products()
        }
    }
}
```

`body` should describe current UI from current state. Loading belongs in a task, model, or view model.

## Conditional Views

SwiftUI conditionals are still strongly typed through builders.

```swift
var body: some View {
    if isLoading {
        ProgressView()
    } else {
        ProductList(products: products)
    }
}
```

A senior engineer thinks about whether switching branches changes identity and state. Replacing a subtree can reset state below it.

## Extracting Subviews

Extract when it improves naming, testability, previewability, or reduces repeated layout.

```swift
struct ProductRow: View {
    let product: Product

    var body: some View {
        HStack {
            VStack(alignment: .leading) {
                Text(product.name)
                Text(product.category)
                    .font(.caption)
                    .foregroundStyle(.secondary)
            }

            Spacer()

            Text(product.price.formatted(.currency(code: "USD")))
        }
    }
}
```

Avoid extracting every three lines mechanically. The goal is understandable composition.

## Type Erasure with `AnyView`

`AnyView` hides concrete view type at runtime.

Use sparingly:

```swift
func destination(for route: Route) -> AnyView {
    switch route {
    case .profile:
        AnyView(ProfileView())
    case .settings:
        AnyView(SettingsView())
    }
}
```

Most code can avoid `AnyView` by using `@ViewBuilder`:

```swift
@ViewBuilder
func destination(for route: Route) -> some View {
    switch route {
    case .profile:
        ProfileView()
    case .settings:
        SettingsView()
    }
}
```

Senior preference: avoid `AnyView` unless the abstraction boundary genuinely needs type erasure.

## Latest SwiftUI Notes

Recent SwiftUI releases continue moving toward:

- Better result-builder consistency through `ContentBuilder`.
- More predictable state initialization behavior.
- Stronger integration with Observation.
- Better performance diagnostics and compile-time behavior.

For fundamentals, the important takeaway is that the `View` protocol is still the center of SwiftUI: value-based descriptions, declarative state, strong static typing, and framework-managed rendering.

## Senior iOS Engineer Perspective

A senior engineer treats `View` design as architecture, not just UI syntax.

Ask:

- Is this view a pure rendering layer?
- Where does state live?
- Is the view doing work that belongs in a model or service?
- Are subviews named around domain meaning?
- Will identity changes reset state unexpectedly?
- Are dependencies injected clearly?
- Can I preview and test this behavior?

## Interview Notes

Junior:

`View` is a protocol. A SwiftUI view describes UI in its `body`.

Mid-level:

Views are value types. SwiftUI recreates them often, so state should live in property wrappers or models, not in transient stored calculations.

Senior:

The `View` protocol is a typed declarative contract. I design views to be cheap descriptions of UI derived from explicit state, keep side effects out of `body`, use identity intentionally, and extract subviews around real ownership and reuse boundaries.

## Practice

1. Build a `ProductRow` view with name, category, price, and selected state.
2. Extract a reusable `EmptyStateView`.
3. Replace an `AnyView` route helper with an `@ViewBuilder` helper.
4. Explain why network calls do not belong directly in `body`.
