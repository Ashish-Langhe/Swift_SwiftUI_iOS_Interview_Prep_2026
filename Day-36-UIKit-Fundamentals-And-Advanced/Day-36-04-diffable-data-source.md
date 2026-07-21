# Day 36.4: Diffable Data Source

Diffable data source is the modern UIKit approach for updating table and collection views. Instead of manually calling `insertRows`, `deleteRows`, and `reloadRows`, you describe the current state with a snapshot. UIKit computes the difference and updates the UI.

## Why Diffable Exists

Old approach:

```swift
items.remove(at: index)
tableView.deleteRows(at: [IndexPath(row: index, section: 0)], with: .automatic)
```

This is easy to break. If your model and table updates disagree, the app can crash with invalid update errors.

Diffable approach:

```swift
var snapshot = NSDiffableDataSourceSnapshot<Section, Item>()
snapshot.appendSections([.main])
snapshot.appendItems(items)
dataSource.apply(snapshot, animatingDifferences: true)
```

The snapshot is the source of truth for what the list should display.

## Basic Collection View Example

```swift
enum Section {
    case main
}

struct ProductRow: Hashable {
    let id: UUID
    let title: String
    let price: String
}

final class ProductGridViewController: UIViewController {
    private var collectionView: UICollectionView!
    private var dataSource: UICollectionViewDiffableDataSource<Section, ProductRow>!

    override func viewDidLoad() {
        super.viewDidLoad()
        configureCollectionView()
        configureDataSource()
    }

    private func configureDataSource() {
        let registration = UICollectionView.CellRegistration<UICollectionViewListCell, ProductRow> { cell, _, row in
            var content = UIListContentConfiguration.subtitleCell()
            content.text = row.title
            content.secondaryText = row.price
            cell.contentConfiguration = content
        }

        dataSource = UICollectionViewDiffableDataSource<Section, ProductRow>(
            collectionView: collectionView
        ) { collectionView, indexPath, row in
            collectionView.dequeueConfiguredReusableCell(
                using: registration,
                for: indexPath,
                item: row
            )
        }
    }

    func render(rows: [ProductRow]) {
        var snapshot = NSDiffableDataSourceSnapshot<Section, ProductRow>()
        snapshot.appendSections([.main])
        snapshot.appendItems(rows, toSection: .main)
        dataSource.apply(snapshot, animatingDifferences: true)
    }
}
```

## Identity Matters

Diffable data source relies on `Hashable`. The identity and equality semantics of your item model affect animations.

Bad:

```swift
struct ProductRow: Hashable {
    let title: String
    let price: String
}
```

If two products share the same title and price, diffable may treat them as the same item.

Better:

```swift
struct ProductRow: Hashable {
    let id: UUID
    let title: String
    let price: String

    func hash(into hasher: inout Hasher) {
        hasher.combine(id)
    }

    static func == (lhs: ProductRow, rhs: ProductRow) -> Bool {
        lhs.id == rhs.id
    }
}
```

Senior nuance: identity-only equality can require explicit reconfigure/reload when display fields change.

## Updating Changed Items

For changed visible content:

```swift
var snapshot = dataSource.snapshot()
snapshot.reconfigureItems([updatedRow])
dataSource.apply(snapshot, animatingDifferences: true)
```

Use `reconfigureItems` for content changes when identity is the same. It is usually lighter than deleting and reinserting.

## Multiple Sections

```swift
enum Section: Hashable {
    case featured
    case recommended
    case recent
}

func apply(home: HomeDisplayModel) {
    var snapshot = NSDiffableDataSourceSnapshot<Section, ProductRow>()
    snapshot.appendSections([.featured, .recommended, .recent])
    snapshot.appendItems(home.featured, toSection: .featured)
    snapshot.appendItems(home.recommended, toSection: .recommended)
    snapshot.appendItems(home.recent, toSection: .recent)
    dataSource.apply(snapshot, animatingDifferences: true)
}
```

## Section Snapshots

Section snapshots are useful for expandable/collapsible hierarchical data.

```swift
var sectionSnapshot = NSDiffableDataSourceSectionSnapshot<MenuItem>()
sectionSnapshot.append([settings])
sectionSnapshot.append(settings.children, to: settings)
sectionSnapshot.expand([settings])

dataSource.apply(sectionSnapshot, to: .main, animatingDifferences: true)
```

Use cases:

- Settings with nested rows.
- File browsers.
- FAQ sections.
- Category trees.

## Apply On Main Actor

UIKit UI work must happen on the main actor.

```swift
@MainActor
func render(_ rows: [ProductRow]) {
    var snapshot = NSDiffableDataSourceSnapshot<Section, ProductRow>()
    snapshot.appendSections([.main])
    snapshot.appendItems(rows)
    dataSource.apply(snapshot, animatingDifferences: true)
}
```

With Swift concurrency, keep data fetching off the main actor but UI updates on it.

## Snapshot Architecture

For senior-level code, build snapshots from screen state.

```swift
enum ProductsState {
    case loading
    case empty
    case loaded([ProductRow])
    case failed(String)
}

func makeSnapshot(for state: ProductsState) -> NSDiffableDataSourceSnapshot<Section, Row> {
    var snapshot = NSDiffableDataSourceSnapshot<Section, Row>()
    snapshot.appendSections([.main])

    switch state {
    case .loading:
        snapshot.appendItems([.loading])
    case .empty:
        snapshot.appendItems([.empty])
    case .loaded(let rows):
        snapshot.appendItems(rows.map(Row.product))
    case .failed(let message):
        snapshot.appendItems([.error(message)])
    }

    return snapshot
}
```

This avoids spreading list update decisions around the controller.

## When To Use `reloadData`

Use `reloadData` or `applySnapshotUsingReloadData` when:

- Initial load is huge and animation is not needed.
- You need to reset broken UI state.
- You are replacing the entire dataset after a major filter change.
- You intentionally do not want diff animations.

Do not use it as the default because it loses fine-grained animation and may reset scroll-related state.

## Common Mistakes

- Using unstable identifiers.
- Including mutable fields in hash identity unintentionally.
- Applying snapshots from background threads.
- Applying many snapshots rapidly for every keystroke without debouncing.
- Mutating backing arrays and snapshots independently.
- Forgetting supplementary views for headers.

## Junior-Level Interview Answer

Diffable data source updates table or collection views by applying snapshots. A snapshot contains sections and items. UIKit compares the old and new snapshots and animates changes safely.

## Senior-Level Interview Answer

Diffable data source makes UI updates state-driven. I design stable item identities, build snapshots from view state, use `reconfigureItems` for content updates, avoid snapshot storms, keep apply calls on the main actor, and choose reload-style applies when diffing would be wasteful or visually noisy.

## Points To Remember

- Snapshot equals current UI state.
- Stable identity is critical.
- `Hashable` design affects animations.
- `reconfigureItems` is for same-identity content updates.
- Apply snapshots on the main actor.
- Diffable reduces invalid update crashes, but it does not fix poor state modeling.

