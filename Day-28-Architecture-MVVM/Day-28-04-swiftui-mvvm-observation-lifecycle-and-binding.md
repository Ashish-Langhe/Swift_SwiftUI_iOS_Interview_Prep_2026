# Day 28: SwiftUI MVVM, Observation, Lifecycle, And Binding

## MVVM In SwiftUI

SwiftUI already has state-driven rendering. MVVM should cooperate with SwiftUI, not fight it.

Modern SwiftUI MVVM often uses:

- `@Observable` ViewModels
- `@State` for owned ViewModel lifetime
- `@Bindable` for bindings into ViewModel properties
- Environment for app/session dependencies
- `.task` for lifecycle-driven async work

## Owning A ViewModel

If a screen creates and owns the ViewModel:

```swift
struct ProductsScreen: View {
    @State private var viewModel: ProductsViewModel

    init(service: ProductService) {
        _viewModel = State(initialValue: ProductsViewModel(service: service))
    }

    var body: some View {
        ProductsContent(viewModel: viewModel)
    }
}
```

Do not create it inside `body`.

Bad:

```swift
var body: some View {
    ProductsContent(viewModel: ProductsViewModel(service: service))
}
```

This can recreate state unexpectedly.

## Passing A ViewModel

If a parent owns it, pass it down.

```swift
struct ProductsContent: View {
    let viewModel: ProductsViewModel

    var body: some View {
        List(viewModel.products) { product in
            ProductRow(product: product)
        }
    }
}
```

With Observation, a plain property can be enough for reading observable state.

## Binding Into Observable ViewModel

Use `@Bindable` when the view needs `$viewModel.property`.

```swift
struct LoginForm: View {
    @Bindable var viewModel: LoginViewModel

    var body: some View {
        Form {
            TextField("Email", text: $viewModel.email)
            SecureField("Password", text: $viewModel.password)

            Button("Sign In") {
                Task { await viewModel.signIn() }
            }
            .disabled(!viewModel.canSubmit)
        }
    }
}
```

## Lifecycle With `.task`

```swift
struct ProductsScreen: View {
    @State private var viewModel: ProductsViewModel

    var body: some View {
        ProductsContent(viewModel: viewModel)
            .task {
                await viewModel.loadIfNeeded()
            }
    }
}
```

Make `loadIfNeeded` idempotent.

```swift
func loadIfNeeded() async {
    guard case .idle = state else { return }
    await load()
}
```

## Input-Driven Loading

```swift
.task(id: productID) {
    await viewModel.load(productID: productID)
}
```

Use when a screen's identity/input changes.

## SwiftUI ViewModel vs UIKit ViewModel

UIKit ViewModels often expose closures, delegates, or reactive streams.

SwiftUI ViewModels expose observable state directly.

UIKit style:

```swift
viewModel.onStateChanged = { [weak self] state in
    self?.render(state)
}
```

SwiftUI style:

```swift
Text(viewModel.title)
```

SwiftUI observes the dependency and updates automatically.

## When Not To Use A ViewModel In SwiftUI

No ViewModel needed:

```swift
struct ProductRow: View {
    let product: Product

    var body: some View {
        Text(product.name)
    }
}
```

Adding `ProductRowViewModel` is useful only if formatting/logic is meaningful or reused.

## Environment ViewModels

Use environment for app-level models.

```swift
@Observable
@MainActor
final class SessionViewModel {
    private(set) var user: User?

    func signOut() {
        user = nil
    }
}
```

Inject:

```swift
RootView()
    .environment(session)
```

Read:

```swift
@Environment(SessionViewModel.self) private var session
```

Avoid putting all feature state into one environment object.

## Common Mistakes

- Recreating ViewModels in `body`.
- Using `@Bindable` everywhere when reading is enough.
- Putting every ViewModel in environment.
- Doing async loading in initializers.
- Treating `.task` as guaranteed once-only.
- ViewModel state reset due to identity changes.

## Senior iOS Engineer Artifact

```text
Artifact: SwiftUI MVVM Lifecycle Review
Screen:
ViewModel owner:
Creation point:
Lifetime:
Bindings needed:
Task trigger:
Idempotent loading:
Input identity:
Environment usage:
State reset risks:
```

## Interview Notes

Junior:

SwiftUI views show ViewModel state and call ViewModel methods.

Mid-level:

Use `@State` to own observable ViewModels and `@Bindable` when a form needs bindings.

Senior:

I design SwiftUI MVVM around ownership and lifetime. I avoid creating ViewModels in `body`, make tasks idempotent, use bindings only for editing, and keep environment models scoped.

## Practice

1. Create a SwiftUI screen that owns a ViewModel with `@State`.
2. Add `@Bindable` to a login form.
3. Make `.task` loading idempotent.
4. Explain why creating a ViewModel in `body` is risky.
