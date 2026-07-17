# Day 28: MVVM Purpose And Mental Model

## What MVVM Is

MVVM means Model-View-ViewModel.

It is an architectural pattern that separates:

- Model: domain data, business rules, services, persistence, API DTOs, repositories.
- View: UI rendering and user interaction.
- ViewModel: presentation state, formatting, state transitions, and user intents.

Simple picture:

```text
View -> user action -> ViewModel -> service/model
View <- observable state <- ViewModel <- result
```

MVVM is not about adding more files. It is about making UI behavior easier to reason about, test, and change.

## Basic Example

Model:

```swift
struct Product: Identifiable, Equatable {
    let id: UUID
    let name: String
    let price: Decimal
}
```

Service:

```swift
protocol ProductService {
    func products() async throws -> [Product]
}
```

ViewModel:

```swift
@Observable
@MainActor
final class ProductsViewModel {
    enum State: Equatable {
        case idle
        case loading
        case loaded([Product])
        case failed(String)
    }

    var state: State = .idle

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

View:

```swift
struct ProductsScreen: View {
    @State private var viewModel: ProductsViewModel

    var body: some View {
        content
            .task {
                await viewModel.load()
            }
    }

    @ViewBuilder
    private var content: some View {
        switch viewModel.state {
        case .idle, .loading:
            ProgressView()
        case .loaded(let products):
            List(products) { product in
                Text(product.name)
            }
        case .failed(let message):
            ContentUnavailableView("Error", systemImage: "exclamationmark.triangle", description: Text(message))
        }
    }
}
```

## Why MVVM Exists

Without separation, views become responsible for everything:

- Fetching data
- Mapping API responses
- Handling errors
- Formatting dates/prices
- Validating forms
- Deciding navigation
- Showing loading states
- Retrying requests
- Tracking analytics

This becomes hard to test and review.

MVVM moves behavior and state transitions into a testable object.

## What MVVM Is Not

MVVM is not:

- A rule that every view needs a ViewModel.
- A replacement for domain modeling.
- A place to dump networking, storage, navigation, analytics, and formatting all together.
- A guarantee of good architecture.
- A SwiftUI requirement.

Bad MVVM can be worse than no MVVM.

## Beginner Mental Model

The View asks:

```text
What should I show?
What happened when the user tapped?
```

The ViewModel answers:

```text
Here is the current state.
Here is what happens for this user intent.
```

The Model/Service answers:

```text
Here is the business data or operation result.
```

## Senior Mental Model

A senior engineer thinks in boundaries:

- ViewModel owns presentation state, not UI layout.
- Services own external effects.
- Domain models own business meaning.
- Views render and send intents.
- Dependencies are injected.
- State transitions are testable.
- Navigation ownership is deliberate.

## Real iOS Use Cases

MVVM fits well for:

- Login screen
- Product list
- Checkout flow
- Search screen
- Settings form
- Profile editor
- Dashboard with multiple async sections
- Detail screen with retry/error/loading states

MVVM may be unnecessary for:

- Simple row views
- Static labels
- Pure formatting-only views
- Tiny reusable components with no behavior

## Decision Rule

Use MVVM when a screen has meaningful behavior:

- Async loading
- Validation
- Multiple UI states
- Error recovery
- Business decisions
- Non-trivial formatting
- Navigation decisions
- Test requirements

Skip MVVM when the view is already a clear pure rendering component.

## Senior iOS Engineer Artifact

```text
Artifact: MVVM Fit Check
Feature:
Does it load data?
Does it validate input?
Does it handle errors?
Does it have multiple states?
Does it need tests?
Does it coordinate services?
Does it navigate?
Is a ViewModel clarifying or adding ceremony?
Decision:
```

## Common Mistakes

- Creating ViewModels for every tiny view.
- Putting network calls directly in SwiftUI views.
- Making ViewModels know too much about UI layout.
- ViewModels directly creating live services.
- One giant app-wide ViewModel.
- No clear state enum.
- No tests for important transitions.

## Interview Notes

Junior:

MVVM separates UI from logic using Model, View, and ViewModel.

Mid-level:

The ViewModel exposes state and handles user actions. The View renders state and forwards actions.

Senior:

I use MVVM selectively where it improves ownership, testability, async state handling, and dependency boundaries. I avoid ceremony for simple views and prevent ViewModels from becoming god objects.

## Practice

1. Identify Model, View, and ViewModel in a login screen.
2. Decide whether a `ProductRow` needs a ViewModel.
3. Convert a view with network loading into MVVM.
4. Explain why MVVM is not automatically good architecture.
