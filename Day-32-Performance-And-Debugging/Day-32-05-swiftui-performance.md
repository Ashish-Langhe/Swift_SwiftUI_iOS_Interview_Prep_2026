# Day 32: SwiftUI Performance

## Why SwiftUI Performance Is Different

SwiftUI performance issues often come from:

- Long view body updates
- Too many view updates
- Unstable identity
- Expensive computed properties
- Large state observed too broadly
- Layout churn
- Heavy work on Main Actor
- Inefficient lists/grids

Apple's SwiftUI performance guidance focuses on hangs, hitches, long body updates, and frequently occurring updates.

## Long Body Updates

Bad:

```swift
var body: some View {
    List(products.sorted { expensiveRank($0) > expensiveRank($1) }) { product in
        ProductRow(product: product)
    }
}
```

`body` should describe UI, not perform expensive work.

Better:

```swift
@Observable
@MainActor
final class ProductsModel {
    var rows: [ProductRowViewModel] = []

    func update(products: [Product]) async {
        rows = await ProductRowMapper.rows(from: products)
    }
}
```

## Frequent Updates

This can be worse than one slow update.

Example:

```swift
@Observable
final class AppModel {
    var scrollOffset: CGFloat = 0
    var user: User?
    var products: [Product] = []
    var cart: Cart = .empty
}
```

If many views observe `AppModel`, changing `scrollOffset` can invalidate too much UI.

Better:

- Keep scroll state local.
- Scope feature models.
- Avoid global observable junk drawers.

## Identity

Bad:

```swift
struct Product: Identifiable {
    var id: UUID { UUID() }
    let name: String
}
```

This destroys identity every update.

Good:

```swift
struct Product: Identifiable {
    let id: UUID
    let name: String
}
```

Stable identity supports diffing, animations, state preservation, and performance.

## List Row Design

Good row:

```swift
struct ProductRow: View {
    let row: ProductRowViewModel

    var body: some View {
        HStack {
            VStack(alignment: .leading) {
                Text(row.title)
                Text(row.subtitle)
                    .foregroundStyle(.secondary)
            }

            Spacer()
            Text(row.priceText)
        }
    }
}
```

Avoid row bodies that:

- Decode images
- Format many dates
- Sort/filter
- Run regex
- Access database
- Start tasks repeatedly

## SwiftUI Instruments

Use SwiftUI profiling tools to find:

- Long view body updates
- Long platform view updates
- Frequent update causes
- Hitches
- Hangs

Workflow:

```text
1. Reproduce hitch
2. Select hitch interval
3. Inspect long body updates
4. Inspect update causes
5. Reduce work or update frequency
6. Re-profile
```

## Main Actor Inheritance

SwiftUI code often runs on Main Actor. If a task is created from SwiftUI and inherits Main Actor context, CPU work can compete with UI updates.

Bad:

```swift
Button("Generate Thumbnails") {
    Task {
        thumbnails = await renderer.renderAll(images)
    }
}
```

If rendering is CPU-heavy and actor-inherited, UI can hang.

Better approach:

- Move CPU-heavy work to non-main isolated code.
- Use `@concurrent` where appropriate in Swift 6.2+.
- Keep UI mutation on Main Actor.

## Layout Performance

Watch for:

- Nested geometry readers
- Constant preference updates
- Huge lazy stacks with heavy rows
- Custom layouts measuring too much
- Dynamic type causing repeated layout churn

Measure before rewriting.

## Common Mistakes

- Blaming SwiftUI instead of measuring.
- Unstable IDs.
- Giant global observable model.
- Expensive work in body.
- Too much state high in tree.
- Tasks restarted by identity changes.
- No device profiling.

## Senior Artifact

```text
Artifact: SwiftUI Performance Review
Screen:
Symptom:
Long body update:
Frequent update source:
Observed state scope:
Identity stability:
Row cost:
Layout cost:
Main Actor work:
Fix:
Trace verification:
```

## Interview Notes

Junior:

Keep SwiftUI views simple and avoid expensive work in `body`.

Mid-level:

Use stable IDs, lightweight rows, scoped state, and Instruments to find long/frequent updates.

Senior:

I debug SwiftUI performance by measuring hitches, long body updates, update causes, identity stability, state scope, layout cost, and Main Actor contention. I verify every fix with another trace.

## Practice

1. Fix unstable list IDs.
2. Move sorting out of `body`.
3. Split global state into feature models.
4. Profile a scrolling hitch with SwiftUI instruments.
