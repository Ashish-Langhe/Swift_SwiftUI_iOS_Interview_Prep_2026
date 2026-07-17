# Day 26: Sheets

## What Sheets Are

Sheets present temporary modal content.

Use sheets for:

- Filters
- Edit flows
- Sharing
- Add item
- Short forms
- Secondary choices
- Lightweight task completion

```swift
struct ProductsScreen: View {
    @State private var isShowingFilters = false

    var body: some View {
        List(products) { product in
            ProductRow(product: product)
        }
        .toolbar {
            Button("Filters") {
                isShowingFilters = true
            }
        }
        .sheet(isPresented: $isShowingFilters) {
            FiltersView()
        }
    }
}
```

## Boolean Sheets

Boolean sheets are fine for one modal.

```swift
@State private var isShowingAddTask = false

.sheet(isPresented: $isShowingAddTask) {
    AddTaskView()
}
```

Once you have multiple sheets, prefer an enum.

## Enum-Driven Sheets

```swift
enum ActiveSheet: Identifiable {
    case filters
    case editProduct(Product.ID)
    case share(Product.ID)

    var id: String {
        switch self {
        case .filters:
            return "filters"
        case .editProduct(let id):
            return "edit-\(id)"
        case .share(let id):
            return "share-\(id)"
        }
    }
}
```

Use:

```swift
struct ProductsScreen: View {
    @State private var activeSheet: ActiveSheet?

    var body: some View {
        ProductList(
            openFilters: { activeSheet = .filters },
            editProduct: { activeSheet = .editProduct($0) },
            shareProduct: { activeSheet = .share($0) }
        )
        .sheet(item: $activeSheet) { sheet in
            switch sheet {
            case .filters:
                FiltersView()
            case .editProduct(let id):
                EditProductView(productID: id)
            case .share(let id):
                ShareProductView(productID: id)
            }
        }
    }
}
```

This avoids conflicting booleans and makes presentation state explicit.

## Dismiss

Use environment dismiss for local modal dismissal.

```swift
struct FiltersView: View {
    @Environment(\.dismiss) private var dismiss

    var body: some View {
        NavigationStack {
            Form {
                // filters
            }
            .toolbar {
                Button("Done") {
                    dismiss()
                }
            }
        }
    }
}
```

Senior tip: dismissing is UI behavior. Saving, validation, and analytics should be handled intentionally, not hidden inside dismissal.

## Passing Data Back

Use bindings for direct editing:

```swift
.sheet(isPresented: $isShowingFilters) {
    FiltersView(filters: $filters)
}
```

Use callbacks for commands:

```swift
AddTaskView { draft in
    tasks.append(TaskItem(draft: draft))
}
```

Rule:

- Binding = edit parent-owned value.
- Callback = submit an action.

## Presentation Detents

```swift
.sheet(isPresented: $isShowingFilters) {
    FiltersView(filters: $filters)
        .presentationDetents([.medium, .large])
        .presentationDragIndicator(.visible)
}
```

Detents are useful for filters, quick actions, and compact editing.

Do not force a medium detent if the content needs space for accessibility text sizes.

## Preventing Interactive Dismiss

```swift
.interactiveDismissDisabled(hasUnsavedChanges)
```

Use this carefully. If you block dismissal, provide a clear cancel/discard path.

```swift
Button("Cancel", role: .cancel) {
    if hasUnsavedChanges {
        activeAlert = .discardChanges
    } else {
        dismiss()
    }
}
```

## Sheet Navigation

Sheets often need their own `NavigationStack`.

```swift
.sheet(item: $activeSheet) { sheet in
    NavigationStack {
        sheetContent(sheet)
    }
}
```

Keep modal navigation local unless it is part of the app-level route system.

## Latest SwiftUI Notes

Recent SwiftUI releases have expanded presentation behavior, including sheet sizing improvements and richer presentation APIs. The practical senior takeaway is to design sheets as explicit presentation state, not scattered flags.

## Common Mistakes

- Multiple boolean sheet flags that can conflict.
- Passing large mutable models into sheets.
- Forgetting unsaved-change behavior.
- Doing save work only in `onDisappear`.
- No accessibility check for detents.
- Using sheets for flows that should be navigation.

## Senior iOS Engineer Perspective

Before choosing a sheet, ask:

- Is this task temporary and interruptible?
- Does the content need navigation?
- Who owns the presented state?
- How does data return?
- What happens on cancellation?
- Can the user dismiss with unsaved changes?
- Does the presentation adapt on iPad?

## Interview Notes

Junior:

Use `.sheet` to present modal content.

Mid-level:

Use `sheet(item:)` with identifiable enum state for multiple sheets.

Senior:

I model sheets as explicit presentation state, choose bindings or callbacks based on ownership, handle unsaved changes, test adaptive presentation, and avoid using sheets as hidden navigation hacks.

## Practice

1. Replace three boolean sheets with one enum.
2. Add a filter sheet that edits a binding.
3. Add a draft form sheet with discard confirmation.
4. Explain sheet vs navigation push.
