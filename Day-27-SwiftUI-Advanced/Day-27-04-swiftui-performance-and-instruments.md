# Day 27: SwiftUI Performance And Instruments

## Why Performance Is Advanced SwiftUI

SwiftUI performance issues often look like:

- Janky scrolling
- Slow animations
- Delayed navigation
- High CPU during state changes
- Repeated body updates
- Slow representable updates
- Excessive layout work

Advanced engineers measure before guessing.

## Core Rule

`body` should be fast.

Bad:

```swift
var body: some View {
    let sorted = products.sorted { expensiveRank($0) > expensiveRank($1) }

    return List(sorted) { product in
        ProductRow(product: product)
    }
}
```

Better:

```swift
@Observable
final class ProductsModel {
    private(set) var visibleProducts: [Product] = []

    func updateVisibleProducts(from products: [Product]) {
        visibleProducts = products.sorted { $0.rank > $1.rank }
    }
}
```

Compute expensive data outside `body`, cache it, and update it when inputs change.

## Frequent Updates

Not all problems are slow body calculations. Some are too many updates.

Common causes:

- Broad observable models
- Environment changes high in the tree
- Timers updating large views
- Scroll offset state updates
- Animations touching too much state
- Parent views owning state too broadly

## Row Performance

```swift
struct ProductRowViewModel: Identifiable, Equatable {
    let id: Product.ID
    let title: String
    let subtitle: String
    let priceText: String
}
```

Rows should receive display-ready data when formatting is expensive.

```swift
struct ProductRow: View {
    let viewModel: ProductRowViewModel

    var body: some View {
        HStack {
            VStack(alignment: .leading) {
                Text(viewModel.title)
                Text(viewModel.subtitle)
                    .foregroundStyle(.secondary)
            }

            Spacer()
            Text(viewModel.priceText)
        }
    }
}
```

## EquatableView

Use `.equatable()` only when equality is cheap and meaningful.

```swift
ProductRow(viewModel: row)
    .equatable()
```

Do not use it as magic dust. If equality is expensive, it can make performance worse.

## Identity And Performance

Unstable IDs cause churn.

Bad:

```swift
var id: UUID { UUID() }
```

Good:

```swift
let id: UUID
```

Stable identity helps SwiftUI diff updates efficiently.

## Instruments

Use the SwiftUI Instruments template to find:

- Long view body updates
- Long platform view updates
- Other long SwiftUI updates
- Causes of updates
- Hitches and hangs
- Time Profiler evidence

Advanced workflow:

1. Reproduce the slow interaction.
2. Profile with the SwiftUI template.
3. Identify long body updates or frequent updates.
4. Use Time Profiler for expensive code.
5. Change one thing.
6. Re-profile.

## Representable Performance

UIKit/AppKit representables can cause long platform updates.

```swift
struct LegacyMapView: UIViewRepresentable {
    let annotations: [Annotation]

    func updateUIView(_ uiView: MKMapView, context: Context) {
        guard context.coordinator.annotations != annotations else {
            return
        }

        context.coordinator.annotations = annotations
        uiView.removeAnnotations(uiView.annotations)
        uiView.addAnnotations(annotations.map(\.mapAnnotation))
    }
}
```

Avoid resetting platform views on every update if nothing meaningful changed.

## Common Mistakes

- Guessing without Instruments.
- Doing sorting/filtering/date formatting in `body`.
- Observing too much state too high in the tree.
- Unstable IDs.
- Heavy image work on the main thread.
- Updating scroll offset into global state.
- Rebuilding UIKit views unnecessarily.

## Senior iOS Engineer Perspective

Senior performance review asks:

- Which interaction is slow?
- Is SwiftUI doing the work or something else?
- Are body updates long or too frequent?
- Is row identity stable?
- Is expensive work cached?
- Are representables diffing updates?
- Did we re-profile after the fix?

## Interview Notes

Junior:

Keep SwiftUI views simple and avoid expensive work in `body`.

Mid-level:

Use stable IDs, lightweight rows, and move expensive calculations into models.

Senior:

I profile with Instruments, distinguish long body updates from frequent updates, inspect cause-and-effect, optimize only measured bottlenecks, and verify improvements with another trace.

## Practice

1. Move sorting out of a view body.
2. Fix unstable list IDs.
3. Add diffing to a `UIViewRepresentable` update.
4. Explain how the SwiftUI Instruments template helps diagnose hitches.
