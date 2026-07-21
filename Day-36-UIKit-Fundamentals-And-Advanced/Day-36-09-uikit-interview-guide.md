# Day 36.9: UIKit Interview Guide

This guide turns Day 36 into interview-ready answers from junior to senior level. UIKit interviews usually test fundamentals first, then push into architecture, performance, lifecycle timing, modern collection views, and migration with SwiftUI.

## Must-Know Topics

- `UIViewController` lifecycle.
- Programmatic UI and `loadView`.
- Auto Layout anchors, priorities, safe areas, scroll views.
- `UITableView` and `UICollectionView`.
- Cell reuse and `prepareForReuse`.
- Delegates and data sources.
- Diffable data source and snapshots.
- Compositional layout.
- Accessibility and Dynamic Type.
- UIKit plus SwiftUI interoperability.
- Memory management with delegates, closures, and tasks.

## Question: Explain UIViewController Lifecycle

Junior answer:

`viewDidLoad` runs once after the view loads. `viewWillAppear` runs before the screen appears. `viewDidAppear` runs after it appears. `viewWillDisappear` and `viewDidDisappear` run when it leaves. I use them to set up UI, refresh visible state, start or stop work, and clean up.

Senior answer:

I separate lifecycle by intent. `loadView` constructs programmatic views. `viewDidLoad` handles one-time setup and binding. `viewWillAppear` refreshes state that may change while away. `viewIsAppearing` is useful for appearance work with a reliable current environment. `viewDidAppear` is appropriate for analytics, focus, or permission prompts. Disappearance callbacks stop visible-only work, and I use `isMovingFromParent` or `isBeingDismissed` to distinguish temporary disappearance from removal.

## Question: What Causes Cell Reuse Bugs?

Common causes:

- Async image completion updates reused cell.
- Old text/image is not reset.
- Selection state is not configured every time.
- Cell stores domain state.
- Callback captures stale index path.

Bad:

```swift
cell.onTap = {
    self.openItem(at: indexPath)
}
```

Better:

```swift
let itemID = row.id
cell.onTap = { [weak self] in
    self?.openItem(id: itemID)
}
```

Senior answer:

Never capture index paths as durable identity. Index paths change after inserts, deletes, sorting, filtering, or diffable updates. Capture stable item IDs.

## Question: Diffable Data Source vs Manual Updates

Junior answer:

Diffable uses snapshots to update table or collection views. It is safer than manually calling insert and delete methods because UIKit calculates the changes.

Senior answer:

Diffable is best when the UI is a function of state. I design stable identities, build snapshots from screen state, use `reconfigureItems` for same-identity content updates, and avoid applying too many snapshots in rapid succession. Manual updates may still be used for simple legacy code or specialized animations, but diffable is usually safer and more maintainable.

## Question: Table View vs Collection View

Junior answer:

Table view is for vertical lists. Collection view is for grids and more flexible layouts.

Senior answer:

Modern collection views can cover list, grid, carousel, and mixed layouts using compositional layout and list configuration. I still use table views for simple settings or legacy screens when the team codebase already uses them, but for new complex screens I usually prefer collection view because it scales better to mixed sections.

## Question: Explain Auto Layout Priority

Junior answer:

Priority decides which constraints are more important. Required constraints have priority 1000. Lower priority constraints can break if needed.

Senior answer:

Priority is how you express graceful degradation. For example, a price label can have required compression resistance while a product title can truncate. A soft max width or flexible spacing can prevent unsatisfiable layouts across iPad, Dynamic Type, and localization.

## Question: How Do You Debug Auto Layout?

Answer:

I reproduce with the exact device, orientation, Dynamic Type size, and language direction. I inspect the Xcode View Debugger, add identifiers to constraints, use the symbolic breakpoint for unsatisfiable constraints, and check whether constraints are being created repeatedly. For self-sizing cells, I verify the vertical constraint chain from content top to bottom.

## Question: How Do Delegates Avoid Retain Cycles?

Answer:

Delegates are usually weak because the parent owns the child view, and the child points back to the parent. The delegate protocol should inherit from `AnyObject` so the property can be weak.

```swift
protocol PaymentViewDelegate: AnyObject {
    func paymentViewDidTapPay(_ view: PaymentView)
}

final class PaymentView: UIView {
    weak var delegate: PaymentViewDelegate?
}
```

## Question: How Do You Migrate UIKit To SwiftUI?

Junior answer:

Use `UIHostingController` to show SwiftUI in UIKit and `UIViewRepresentable` to show UIKit views in SwiftUI.

Senior answer:

I migrate by feature boundary, not by random views. I keep one navigation owner, one state owner, and clear callback boundaries. UIKit coordinators can present SwiftUI screens through hosting controllers, while SwiftUI screens can wrap UIKit controls through representables. I profile hosted cells and avoid assuming SwiftUI `.onAppear` equals UIKit `viewDidLoad`.

## Senior UIKit Design Artifact

Before implementing a UIKit screen, define:

- Screen states: loading, loaded, empty, failed.
- Data source model: sections and row identifiers.
- Navigation outputs: what user actions leave the screen?
- Ownership: who owns state, controller, child controllers, tasks?
- Layout strategy: table, collection list, grid, compositional sections, or custom view?
- Reuse strategy: cell registration, content configuration, or custom cells?
- Accessibility: labels, traits, Dynamic Type, identifiers.
- Performance risks: images, pagination, self-sizing, diff frequency.

## Real iOS Scenario: Product Feed

Senior implementation shape:

```swift
enum FeedSection: Hashable {
    case featured
    case recommended
}

enum FeedItem: Hashable {
    case hero(ProductRow)
    case product(ProductRow)
    case loading
    case error(String)
}
```

Use:

- Compositional layout for hero carousel plus grid.
- Diffable snapshot for state updates.
- Image loader with cancellation.
- View model for pagination.
- Coordinator for product details.
- Accessibility identifiers for UI tests.

## Red Flags In Interview Answers

- "I put everything in viewDidLoad."
- "I call reloadData whenever anything changes."
- "Cells can handle navigation."
- "Auto Layout is just constraints."
- "SwiftUI replaces UIKit completely."
- "prepareForReuse is enough to fix all reuse bugs."
- "I capture indexPath in cell closures."

## Strong Senior Phrases

- "I separate one-time setup from visibility-driven refresh."
- "I use stable item identity, not index paths, for actions."
- "I build snapshots from view state."
- "I keep UIKit and SwiftUI ownership boundaries explicit."
- "I profile scrolling before optimizing."
- "I test Dynamic Type and right-to-left layouts."
- "I use child containment correctly for hosting controllers."

## Practice Tasks

1. Build a settings screen using collection view list layout.
2. Build a product grid using compositional layout.
3. Convert manual collection updates to diffable snapshots.
4. Wrap a UIKit text field in SwiftUI using `UIViewRepresentable`.
5. Embed a SwiftUI profile header into a UIKit screen.
6. Debug an Auto Layout warning from a self-sizing cell.
7. Fix a reused cell that shows the wrong image after fast scrolling.

## Final Interview Checklist

- Can explain lifecycle with examples.
- Can write basic Auto Layout by hand.
- Can explain reuse and async image bugs.
- Can build diffable snapshots.
- Can describe compositional layout hierarchy.
- Can design delegates without retain cycles.
- Can bridge UIKit and SwiftUI both ways.
- Can discuss accessibility and Dynamic Type.
- Can reason about performance in scrolling screens.

