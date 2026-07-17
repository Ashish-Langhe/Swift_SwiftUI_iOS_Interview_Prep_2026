# Day 25: View Identity

## What View Identity Means

Identity tells SwiftUI whether a view is the same logical view across updates.

This matters because identity affects:

- `@State` lifetime
- Animations
- Transitions
- List diffing
- Navigation
- Focus
- Scroll position
- Task cancellation and restart

SwiftUI can recreate view values often. Identity is how it knows what should persist.

## Structural Identity

SwiftUI uses the structure of your view tree to infer identity.

```swift
var body: some View {
    VStack {
        Text("Title")
        CounterView()
    }
}
```

If `CounterView` remains in the same position and same structural branch, SwiftUI generally treats it as the same logical view.

## Explicit Identity with `id`

You can provide explicit identity:

```swift
UserProfileView(user: user)
    .id(user.id)
```

Changing the `id` tells SwiftUI this is a new logical view. That can reset state below it.

## Real Example: Resetting Form State

```swift
struct EditUserScreen: View {
    let user: User

    var body: some View {
        EditUserForm(user: user)
            .id(user.id)
    }
}
```

When `user.id` changes, `EditUserForm` receives a fresh identity, so local draft state resets.

This can be correct if changing users should discard the old draft.

## Identity in Lists

```swift
struct Product: Identifiable {
    let id: UUID
    var name: String
}

List(products) { product in
    ProductRow(product: product)
}
```

Stable IDs let SwiftUI understand insertions, deletions, moves, and row updates.

Avoid unstable IDs:

```swift
struct BadProduct: Identifiable {
    var id: UUID { UUID() }
    let name: String
}
```

This creates a new identity every time and can break diffing, animations, state, and performance.

## `ForEach` Identity

Good:

```swift
ForEach(products, id: \.id) { product in
    ProductRow(product: product)
}
```

Risky:

```swift
ForEach(products.indices, id: \.self) { index in
    ProductRow(product: products[index])
}
```

Index identity can break when items reorder or delete. Use real model identity when possible.

## Conditional Identity

This can reset state:

```swift
if isEditing {
    ProfileForm(mode: .edit)
} else {
    ProfileForm(mode: .view)
}
```

These are separate structural branches. If preserving state matters, keep one view and vary inputs:

```swift
ProfileForm(mode: isEditing ? .edit : .view)
```

## Identity and `.task`

Tasks can restart when view identity changes.

```swift
ProductDetailsView(productID: productID)
    .task(id: productID) {
        await model.load(productID: productID)
    }
```

This is intentional. When `productID` changes, cancel old work and start new work.

## Identity and Animation

Stable identity makes animations meaningful.

```swift
withAnimation {
    products.move(fromOffsets: source, toOffset: destination)
}
```

If item IDs are stable, SwiftUI can animate movement. If IDs change, it may animate removal and insertion instead.

## Latest SwiftUI Notes

Recent SwiftUI updates include more container reordering APIs and broader interaction support. These features depend heavily on stable identity. If a senior engineer cannot explain item identity, drag reordering and animated lists become fragile quickly.

## Common Mistakes

- Using `UUID()` computed properties for IDs.
- Using array indices as identity for mutable lists.
- Adding `.id(...)` to force refreshes without understanding state reset.
- Splitting conditional branches and accidentally losing state.
- Changing identity while async tasks are running.

## Senior iOS Engineer Perspective

Senior SwiftUI engineers debug many "random" UI bugs by asking identity questions:

- Is this the same logical item?
- Is the ID stable across refreshes?
- Should this state survive this change?
- Is the view branch changing?
- Does `.task(id:)` restart intentionally?
- Does the navigation path use stable values?

Identity is not just syntax. It is how SwiftUI connects past UI to future UI.

## Interview Notes

Junior:

Identity helps SwiftUI know which views and list items are the same after state changes.

Mid-level:

Stable `Identifiable` models are important for `List`, `ForEach`, animations, and preserving state.

Senior:

I treat identity as part of UI correctness. I avoid unstable IDs, use explicit `.id` only when I want reset semantics, and design navigation, tasks, lists, and animations around stable domain identity.

## Practice

1. Fix a list that uses `UUID()` as a computed ID.
2. Convert index-based `ForEach` to model identity.
3. Use `.task(id:)` to reload details when selected ID changes.
4. Explain why `.id(user.id)` can reset form state.
