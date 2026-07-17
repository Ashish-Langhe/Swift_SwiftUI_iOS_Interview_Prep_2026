# Day 26: Lists

## What `List` Is

`List` is SwiftUI's high-level scrolling container for row-based content.

Use it for:

- Settings screens
- Search results
- Inbox-style screens
- Master lists
- Menus
- Grouped content
- Editable rows
- Selection-based interfaces

```swift
struct ProductsListView: View {
    let products: [Product]

    var body: some View {
        List(products) { product in
            ProductRow(product: product)
        }
        .navigationTitle("Products")
    }
}
```

`List` is not just a vertical stack. It is platform-adaptive. On iOS, it behaves like a table-style scrolling view. On macOS and iPad, it can participate in sidebar and selection patterns.

## Stable Identity Is Non-Negotiable

Rows need stable identity.

```swift
struct Product: Identifiable, Hashable {
    let id: UUID
    var name: String
    var price: Decimal
}
```

Good:

```swift
List(products) { product in
    ProductRow(product: product)
}
```

Avoid:

```swift
struct Product: Identifiable {
    var id: UUID { UUID() }
    let name: String
}
```

That computed `UUID()` changes on every access and breaks diffing, animations, selection, and row state.

## Sections

Sections communicate hierarchy and improve scanability.

```swift
List {
    Section("Available") {
        ForEach(availableProducts) { product in
            ProductRow(product: product)
        }
    }

    Section("Out of Stock") {
        ForEach(outOfStockProducts) { product in
            ProductRow(product: product)
        }
    }
}
```

Senior tip: sectioning should reflect user workflow, not just backend data shape.

## Row Actions

Use swipe actions for common row-level commands.

```swift
List(products) { product in
    ProductRow(product: product)
        .swipeActions(edge: .trailing) {
            Button(role: .destructive) {
                delete(product)
            } label: {
                Label("Delete", systemImage: "trash")
            }
        }
        .swipeActions(edge: .leading) {
            Button {
                favorite(product)
            } label: {
                Label("Favorite", systemImage: "heart")
            }
            .tint(.pink)
        }
}
```

Keep destructive actions explicit and recoverable where possible.

## Selection

Selection is useful for iPad/macOS, edit mode, and master-detail flows.

```swift
struct ProductPickerView: View {
    let products: [Product]
    @State private var selection: Product.ID?

    var body: some View {
        List(products, selection: $selection) { product in
            Text(product.name)
                .tag(product.id)
        }
    }
}
```

Senior tip: separate "selected item" from "navigation route" when the app supports split view, keyboard navigation, or deep links.

## Moving and Deleting Rows

```swift
struct EditableCategoriesView: View {
    @State private var categories: [Category] = Category.defaults

    var body: some View {
        List {
            ForEach(categories) { category in
                Text(category.name)
            }
            .onMove { source, destination in
                categories.move(fromOffsets: source, toOffset: destination)
            }
            .onDelete { offsets in
                categories.remove(atOffsets: offsets)
            }
        }
        .toolbar {
            EditButton()
        }
    }
}
```

Persist order changes deliberately. Moving rows only changes local state unless you save it.

## Searchable Lists

```swift
struct OrdersView: View {
    @State private var query = ""
    let orders: [Order]

    var visibleOrders: [Order] {
        orders.filter {
            query.isEmpty ||
            $0.customerName.localizedCaseInsensitiveContains(query)
        }
    }

    var body: some View {
        List(visibleOrders) { order in
            OrderRow(order: order)
        }
        .searchable(text: $query, prompt: "Search orders")
    }
}
```

For large datasets, filtering should move to a model or service and may need debouncing.

## Loading, Empty, Error States

A list screen needs all states, not just the happy path.

```swift
enum ProductsState {
    case loading
    case loaded([Product])
    case empty
    case failed(String)
}

struct ProductsScreen: View {
    let state: ProductsState

    var body: some View {
        switch state {
        case .loading:
            ProgressView()
        case .loaded(let products):
            List(products) { ProductRow(product: $0) }
        case .empty:
            ContentUnavailableView("No Products", systemImage: "shippingbox")
        case .failed(let message):
            ContentUnavailableView("Unable to Load", systemImage: "exclamationmark.triangle", description: Text(message))
        }
    }
}
```

Senior engineers design these states before implementation, not after QA reports blank screens.

## Latest SwiftUI Notes

Recent SwiftUI updates expand reordering beyond classic `List` patterns, including newer reorderable container APIs. The senior-level takeaway is that stable identity and predictable item mutation matter even more when drag-and-drop reordering, grids, stacks, and custom layouts all share interaction patterns.

## Performance Tips

- Keep row views lightweight.
- Do not perform formatting-heavy work repeatedly in `body`.
- Prefer precomputed row view models for expensive display logic.
- Use stable IDs.
- Avoid nested scroll views inside rows.
- Avoid storing row-local state if identity can change.
- Test with realistic item counts.

## Senior iOS Engineer Perspective

For a production `List`, ask:

- Are IDs stable across refreshes?
- Does selection survive data reloads?
- Are loading, empty, and error states explicit?
- Are row actions discoverable and safe?
- Does the list support dynamic type?
- Does search scale?
- Is row rendering cheap?
- Are reorder/delete mutations persisted?

## Interview Notes

Junior:

`List` displays rows of data, often using `Identifiable` models.

Mid-level:

Lists need stable IDs, sections, row actions, selection, search, and proper empty/error states.

Senior:

I treat `List` as a stateful, identity-driven container. I design for diffing, accessibility, row performance, selection, persistence of edits, and platform-adaptive behavior.

## Practice

1. Build a sectioned list for available and unavailable products.
2. Add swipe actions for favorite and delete.
3. Add search with a computed filtered list.
4. Refactor unstable IDs into stable domain IDs.
