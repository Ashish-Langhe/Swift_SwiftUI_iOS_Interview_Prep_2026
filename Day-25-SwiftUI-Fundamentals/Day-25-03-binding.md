# Day 25: Binding

## What `Binding` Means

A `Binding<Value>` is a two-way connection to state owned somewhere else.

Parent owns:

```swift
struct SettingsView: View {
    @State private var notificationsEnabled = true

    var body: some View {
        NotificationToggle(isOn: $notificationsEnabled)
    }
}
```

Child edits:

```swift
struct NotificationToggle: View {
    @Binding var isOn: Bool

    var body: some View {
        Toggle("Notifications", isOn: $isOn)
    }
}
```

The child does not own the value. It receives read/write access.

## When to Use `@Binding`

Use binding when:

- Parent owns the source of truth.
- Child needs to edit it.
- You want a reusable control.
- You do not want the child to create independent state.

Examples:

- `TextField`
- `Toggle`
- `Picker`
- Sheet presentation
- Custom filter controls
- Editable rows

## Source of Truth

A binding is not the source of truth. It is a reference-like connection to a source of truth.

```swift
@State private var selectedCategory: Category?

CategoryPicker(selection: $selectedCategory)
```

`selectedCategory` is the state. `$selectedCategory` is the binding.

## Binding to Nested Values

Bindings can target nested mutable state.

```swift
struct ProfileDraft {
    var name: String
    var acceptsMarketing: Bool
}

struct ProfileForm: View {
    @Binding var draft: ProfileDraft

    var body: some View {
        TextField("Name", text: $draft.name)
        Toggle("Marketing emails", isOn: $draft.acceptsMarketing)
    }
}
```

This is useful for form components.

## Custom Bindings

You can create a binding with explicit get/set logic.

```swift
struct QuantityStepper: View {
    @Binding var quantity: Int

    var body: some View {
        Stepper(
            "Quantity: \(quantity)",
            value: Binding(
                get: { quantity },
                set: { quantity = max(1, min($0, 99)) }
            )
        )
    }
}
```

This clamps invalid values.

## Real iOS Example: Search Filter Sheet

```swift
struct ProductsScreen: View {
    @State private var filters = ProductFilters()
    @State private var isShowingFilters = false

    var body: some View {
        ProductList(filters: filters)
            .toolbar {
                Button("Filters") {
                    isShowingFilters = true
                }
            }
            .sheet(isPresented: $isShowingFilters) {
                FiltersView(filters: $filters)
            }
    }
}

struct FiltersView: View {
    @Binding var filters: ProductFilters

    var body: some View {
        Form {
            Toggle("Only available", isOn: $filters.onlyAvailable)
            Picker("Sort", selection: $filters.sort) {
                Text("Name").tag(ProductSort.name)
                Text("Price").tag(ProductSort.price)
            }
        }
    }
}
```

The sheet edits state owned by the products screen.

## Binding vs Callback

Binding is good for direct two-way state editing.

Callback is better for commands or business actions.

```swift
struct FavoriteButton: View {
    let isFavorite: Bool
    let toggleFavorite: () -> Void

    var body: some View {
        Button {
            toggleFavorite()
        } label: {
            Image(systemName: isFavorite ? "heart.fill" : "heart")
        }
    }
}
```

Use a binding if the child edits a value. Use a callback if the child sends an intent.

## Binding and `@Bindable`

With Observation, `@Bindable` creates bindings to mutable properties of an observable model.

```swift
@Observable
final class ProfileModel {
    var name = ""
    var isPublic = false
}

struct ProfileEditor: View {
    @Bindable var model: ProfileModel

    var body: some View {
        TextField("Name", text: $model.name)
        Toggle("Public", isOn: $model.isPublic)
    }
}
```

This is modern SwiftUI data flow for observable objects.

## Common Mistakes

- Creating local `@State` in a child when the parent should own the value.
- Passing a binding too far down many layers.
- Using binding for commands instead of state.
- Mutating domain state from a view without validation.
- Creating custom bindings with surprising side effects.

## Senior iOS Engineer Perspective

A senior engineer uses bindings as a sharp tool:

- Keep source of truth obvious.
- Use bindings for local editing boundaries.
- Prefer callbacks for commands.
- Avoid sending bindings through too many unrelated layers.
- Validate sensitive state changes in models or reducers, not random controls.
- Make reusable components accept bindings only when editing is their purpose.

## Interview Notes

Junior:

`@Binding` lets a child view read and write state owned by a parent.

Mid-level:

Bindings preserve a single source of truth and are commonly passed with `$state`.

Senior:

I use bindings for direct value editing and keep ownership explicit. For business events, I prefer intent callbacks or model methods because they preserve validation and reduce accidental mutation.

## Practice

1. Build a reusable `SearchField` with `@Binding`.
2. Create a custom binding that clamps a number.
3. Replace a binding with a callback where the child performs an action.
4. Explain `@Binding` vs `@Bindable`.
