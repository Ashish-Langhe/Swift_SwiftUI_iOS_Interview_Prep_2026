# Day 25: State

## What State Means in SwiftUI

State is data that can change over time and cause the UI to update.

```swift
struct CounterView: View {
    @State private var count = 0

    var body: some View {
        VStack(spacing: 16) {
            Text("Count: \(count)")

            Button("Increment") {
                count += 1
            }
        }
    }
}
```

When `count` changes, SwiftUI invalidates the relevant view computation and re-evaluates `body`.

## `@State`

Use `@State` for view-owned local state.

Good examples:

- Toggle value
- Selected tab
- Search text
- Local loading flag
- Sheet presentation state
- Temporary form draft

```swift
struct SearchHeader: View {
    @State private var query = ""

    var body: some View {
        TextField("Search products", text: $query)
            .textFieldStyle(.roundedBorder)
    }
}
```

The view owns this value. No parent needs to control it.

## Do Not Use `@State` for Everything

Avoid placing app-wide or business-owned state directly in a leaf view.

Weak design:

```swift
struct ProfileView: View {
    @State private var user = User.empty
}
```

Better design:

```swift
struct ProfileView: View {
    let user: User
}
```

If the parent owns the user, pass it in. If a model owns user loading and mutation, use observable state.

## `@State` and Reference Types

Modern SwiftUI supports using `@State` with observable reference types.

```swift
@Observable
final class ProfileModel {
    var name = ""
    var isSaving = false
}

struct ProfileEditor: View {
    @State private var model = ProfileModel()

    var body: some View {
        Form {
            TextField("Name", text: $model.name)

            Button("Save") {
                model.isSaving = true
            }
        }
    }
}
```

Use this when the view owns the lifetime of the observable model.

## Latest State Initialization Note

Recent SwiftUI updates call out improved state initialization behavior with newer toolchains, including `State()` macro behavior for class-backed state in `App`, `Scene`, and `View`.

Practical meaning:

- SwiftUI continues to make state ownership more predictable.
- You should still avoid relying on view initializers for repeated setup.
- Create state at the ownership boundary and pass data down.

## State Lifetime

`@State` storage is managed by SwiftUI, not by the view struct itself.

```swift
struct LoginView: View {
    @State private var email = ""
    @State private var password = ""

    var body: some View {
        Form {
            TextField("Email", text: $email)
            SecureField("Password", text: $password)
        }
    }
}
```

The `LoginView` value can be recreated, but SwiftUI preserves the state storage as long as the view identity remains stable.

## Real iOS Example: Form Draft

```swift
struct EditProfileView: View {
    let original: Profile
    let onSave: (ProfileDraft) async -> Void

    @State private var draft: ProfileDraft
    @State private var isSaving = false

    init(original: Profile, onSave: @escaping (ProfileDraft) async -> Void) {
        self.original = original
        self.onSave = onSave
        _draft = State(initialValue: ProfileDraft(profile: original))
    }

    var body: some View {
        Form {
            TextField("Display name", text: $draft.displayName)
            Toggle("Public profile", isOn: $draft.isPublic)

            Button("Save") {
                Task {
                    isSaving = true
                    await onSave(draft)
                    isSaving = false
                }
            }
            .disabled(isSaving)
        }
    }
}
```

This is a good `@State` use case because the edit draft belongs to the screen.

## Mistake: Derived State

Do not store values that can be derived cheaply from source state.

Bad:

```swift
@State private var products: [Product] = []
@State private var filteredProducts: [Product] = []
@State private var query = ""
```

Better:

```swift
@State private var products: [Product] = []
@State private var query = ""

var filteredProducts: [Product] {
    products.filter {
        query.isEmpty || $0.name.localizedCaseInsensitiveContains(query)
    }
}
```

Derived state can drift out of sync.

## State and Concurrency

UI state should be mutated on the main actor.

```swift
@MainActor
final class ProductsModel {
    var products: [Product] = []

    func load() async {
        products = await service.products()
    }
}
```

SwiftUI views are main-actor oriented. A senior engineer designs async flows so UI mutations are isolated correctly.

## Senior iOS Engineer Perspective

Senior state design starts with ownership:

- Who creates this data?
- Who can mutate it?
- How long should it live?
- Does it survive navigation?
- Is it local UI state or domain state?
- Can it be derived instead of stored?
- Does async work update it safely?

Bad state ownership creates most SwiftUI bugs: lost form values, stale lists, flickering loading states, accidental resets, and hard-to-test views.

## Interview Notes

Junior:

Use `@State` when a view owns a value that changes and should refresh the UI.

Mid-level:

`@State` storage survives view value recreation as long as identity is stable. Use bindings to pass mutable state to child views.

Senior:

I classify state by ownership and lifetime: local UI state, parent-owned state, observable feature state, environment state, and persisted state. I avoid duplicated derived state, keep async mutations main-actor safe, and test state transitions through user-visible behavior.

## Practice

1. Build a login form with `@State`.
2. Convert duplicated derived state into a computed property.
3. Create an edit draft initialized from an input model.
4. Explain why changing view identity can reset `@State`.
