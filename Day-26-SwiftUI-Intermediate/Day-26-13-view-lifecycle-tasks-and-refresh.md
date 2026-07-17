# Day 26: View Lifecycle, Tasks, And Refresh

## Why This Topic Matters

SwiftUI views are values, but screens still need lifecycle behavior:

- Load data
- Refresh data
- Start and cancel async work
- React to scene phase
- Perform cleanup
- Track analytics

The challenge is doing this without pretending SwiftUI views are long-lived controllers.

## `onAppear`

```swift
struct ProductDetailsView: View {
    let productID: Product.ID

    var body: some View {
        ProductDetailsContent()
            .onAppear {
                analytics.track("product_details_viewed")
            }
    }
}
```

`onAppear` can run more than once. Do not assume it is a one-time initializer.

## `task`

Use `.task` for async loading tied to view lifetime.

```swift
struct ProductsScreen: View {
    @State private var model: ProductsModel

    var body: some View {
        ProductList(products: model.products)
            .task {
                await model.load()
            }
    }
}
```

SwiftUI can cancel the task when the view disappears.

## `task(id:)`

Reload when an input changes.

```swift
struct ProductDetailsScreen: View {
    let productID: Product.ID
    @State private var model: ProductDetailsModel

    var body: some View {
        ProductDetailsContent(state: model.state)
            .task(id: productID) {
                await model.load(productID: productID)
            }
    }
}
```

When `productID` changes, SwiftUI cancels the old task and starts a new one.

## Avoid Duplicate Loading

If `.task` can run more than once, make loading idempotent.

```swift
@Observable
@MainActor
final class ProductsModel {
    enum State {
        case idle
        case loading
        case loaded([Product])
        case failed(String)
    }

    var state: State = .idle

    func loadIfNeeded() async {
        guard case .idle = state else {
            return
        }

        await load()
    }

    func load() async {
        state = .loading
        // fetch
    }
}
```

## Pull To Refresh

```swift
List(model.products) { product in
    ProductRow(product: product)
}
.refreshable {
    await model.reload()
}
```

Refreshing is a user action. It should not be blocked just because initial loading already happened.

## Scene Phase

```swift
struct RootView: View {
    @Environment(\.scenePhase) private var scenePhase
    @State private var session: SessionModel

    var body: some View {
        ContentView()
            .onChange(of: scenePhase) { _, newPhase in
                if newPhase == .active {
                    Task {
                        await session.refreshIfNeeded()
                    }
                }
            }
    }
}
```

Use scene phase for app lifecycle concerns, not routine row updates.

## Analytics

Analytics should be intentional.

```swift
.onAppear {
    analytics.track("checkout_screen_appeared")
}
```

If a view appears multiple times, duplicate events can happen. Consider tracking at navigation boundaries or using a model to control once-per-flow events.

## Cancellation

Async work should cooperate with cancellation.

```swift
func search(query: String) async {
    do {
        try Task.checkCancellation()
        let results = try await service.search(query)
        try Task.checkCancellation()
        self.results = results
    } catch is CancellationError {
        return
    } catch {
        errorMessage = "Search failed."
    }
}
```

Senior engineers consider cancellation part of correctness.

## Common Mistakes

- Treating `onAppear` as a one-time event.
- Creating async work without cancellation awareness.
- Loading the same data repeatedly.
- Ignoring `task(id:)` for input-driven loading.
- Doing analytics in every row's `onAppear` without deduping.
- Mutating UI state from the wrong actor.

## Senior iOS Engineer Perspective

Senior lifecycle thinking:

- What owns loading?
- Is loading idempotent?
- Should this reload when an ID changes?
- What cancels the work?
- What happens when app returns active?
- Are analytics duplicated?
- Are UI mutations main-actor safe?

## Interview Notes

Junior:

Use `.task` for async work and `.onAppear` for simple appear behavior.

Mid-level:

Use `.task(id:)` to restart async work when an input changes and `.refreshable` for pull-to-refresh.

Senior:

I design lifecycle behavior around idempotency, cancellation, actor isolation, scene phase, analytics correctness, and state ownership.

## Practice

1. Replace `onAppear` async loading with `.task`.
2. Add `task(id:)` for a detail screen.
3. Make loading idempotent.
4. Add cancellation checks to a search model.
