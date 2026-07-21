# Day 36.3: UITableView And UICollectionView

`UITableView` and `UICollectionView` are UIKit's core scrolling data presentation views. Interviews often start with table views and then move into collection views, modern list APIs, diffable data sources, reuse, prefetching, performance, empty states, and architecture.

## Table View Mental Model

A table view displays rows in sections. It asks a data source:

- How many sections?
- How many rows in each section?
- Which cell for an index path?

It tells a delegate:

- Row selected.
- Height needed.
- Swipe action requested.
- Will display cell.

Basic example:

```swift
final class SettingsViewController: UIViewController {
    private let tableView = UITableView(frame: .zero, style: .insetGrouped)
    private let rows: [String] = ["Account", "Notifications", "Privacy"]

    override func viewDidLoad() {
        super.viewDidLoad()
        tableView.dataSource = self
        tableView.delegate = self
        tableView.register(UITableViewCell.self, forCellReuseIdentifier: "Cell")
        view.addSubview(tableView)
        tableView.frame = view.bounds
        tableView.autoresizingMask = [.flexibleWidth, .flexibleHeight]
    }
}

extension SettingsViewController: UITableViewDataSource {
    func tableView(_ tableView: UITableView, numberOfRowsInSection section: Int) -> Int {
        rows.count
    }

    func tableView(_ tableView: UITableView, cellForRowAt indexPath: IndexPath) -> UITableViewCell {
        let cell = tableView.dequeueReusableCell(withIdentifier: "Cell", for: indexPath)
        cell.textLabel?.text = rows[indexPath.row]
        cell.accessoryType = .disclosureIndicator
        return cell
    }
}

extension SettingsViewController: UITableViewDelegate {
    func tableView(_ tableView: UITableView, didSelectRowAt indexPath: IndexPath) {
        tableView.deselectRow(at: indexPath, animated: true)
        print("Open \(rows[indexPath.row])")
    }
}
```

## Collection View Mental Model

A collection view displays items using a layout object. It can behave like:

- A grid.
- A list.
- A carousel.
- A Pinterest-like layout.
- A dashboard.
- A sectioned home screen.

Traditional setup:

```swift
let layout = UICollectionViewFlowLayout()
layout.itemSize = CGSize(width: 120, height: 160)
layout.minimumInteritemSpacing = 12
layout.minimumLineSpacing = 12

let collectionView = UICollectionView(frame: .zero, collectionViewLayout: layout)
```

Modern UIKit often uses:

- Diffable data source.
- Cell registrations.
- Compositional layout.
- List configuration.

## Cell Reuse

Cells are reused for performance. You configure the cell for the current item every time.

```swift
final class ProductCell: UICollectionViewCell {
    static let reuseID = "ProductCell"

    private let titleLabel = UILabel()
    private let priceLabel = UILabel()

    override init(frame: CGRect) {
        super.init(frame: frame)
        setup()
    }

    required init?(coder: NSCoder) {
        fatalError("init(coder:) has not been implemented")
    }

    func configure(with product: Product) {
        titleLabel.text = product.name
        priceLabel.text = product.formattedPrice
    }

    override func prepareForReuse() {
        super.prepareForReuse()
        titleLabel.text = nil
        priceLabel.text = nil
    }
}
```

Senior rule: reset transient state in `prepareForReuse`, but do not treat it as the main configuration path. The cell should be correct after `configure`.

## Modern Cell Configuration

For many cells, use `UIListContentConfiguration`.

```swift
var content = UIListContentConfiguration.subtitleCell()
content.text = user.name
content.secondaryText = user.email
content.image = UIImage(systemName: "person.circle")
cell.contentConfiguration = content
```

Benefits:

- Dynamic Type support.
- Standard spacing.
- Better consistency.
- Less custom cell code.

## Data Source Design

Avoid making view controllers responsible for too many transformations.

Better:

```swift
struct ProductRow: Hashable {
    let id: UUID
    let title: String
    let subtitle: String
    let price: String
}

final class ProductListViewModel {
    func rows(from products: [Product]) -> [ProductRow] {
        products.map {
            ProductRow(
                id: $0.id,
                title: $0.name,
                subtitle: $0.categoryName,
                price: $0.formattedPrice
            )
        }
    }
}
```

Then the cell only renders display-ready data.

## Prefetching

Use prefetching for images or data likely needed soon.

```swift
extension FeedViewController: UITableViewDataSourcePrefetching {
    func tableView(_ tableView: UITableView, prefetchRowsAt indexPaths: [IndexPath]) {
        let urls = indexPaths.compactMap { rows[$0.row].imageURL }
        imageLoader.prefetch(urls)
    }

    func tableView(_ tableView: UITableView, cancelPrefetchingForRowsAt indexPaths: [IndexPath]) {
        let urls = indexPaths.compactMap { rows[$0.row].imageURL }
        imageLoader.cancelPrefetch(urls)
    }
}
```

Senior thinking:

- Prefetch should be cancellable.
- Prefetch should not mutate visible UI directly.
- Image loading should handle reuse and cancellation.

## Pagination

Simple table pagination:

```swift
func tableView(_ tableView: UITableView, willDisplay cell: UITableViewCell, forRowAt indexPath: IndexPath) {
    let threshold = rows.index(rows.endIndex, offsetBy: -5)
    if indexPath.row == threshold {
        viewModel.loadNextPage()
    }
}
```

Production concerns:

- Prevent duplicate page requests.
- Track next cursor.
- Handle retry.
- Show loading footer.
- Avoid index assumptions after filtering.

## Empty And Error States

Do not leave a blank list.

```swift
func render(_ state: ProductListState) {
    switch state {
    case .loading:
        backgroundView = LoadingView()
    case .empty:
        backgroundView = EmptyStateView(message: "No products found")
    case .loaded:
        backgroundView = nil
        applySnapshot()
    case .failed(let message):
        backgroundView = ErrorStateView(message: message)
    }
}
```

## Table View vs Collection View

Use table view when:

- You need simple vertical rows.
- The design is mostly list-based.
- Section headers and standard editing behavior are enough.

Use collection view when:

- Layout is grid, carousel, mixed, or adaptive.
- Sections need different layouts.
- You want modern list + grid composition.
- You need more control over supplementary views.

Senior note: modern collection views can replace many table view use cases because compositional layout supports list sections.

## Performance Basics

Good scrolling performance needs:

- Reuse identifiers or registrations.
- Minimal work in `cellForRowAt`.
- Image decoding off main thread.
- Cancellable image requests.
- Estimated sizes for self-sizing cells.
- Avoid synchronous date/number formatting per cell.
- Avoid layout invalidation storms.

Bad cell configuration:

```swift
cell.imageView.image = downloadImageSynchronously(url)
cell.textLabel?.text = expensiveFormatter.string(from: date)
```

Better:

```swift
cell.configure(with: row)
imageLoader.load(row.imageURL) { [weak cell] image in
    cell?.imageView.image = image
}
```

In production, verify the cell still represents the same item before assigning async images.

## Common Interview Traps

- Confusing data source and delegate responsibilities.
- Forgetting cell reuse.
- Mutating data and UI out of sync.
- Using `reloadData` for every small change.
- Doing network or image decoding on the main thread.
- Not handling empty and error states.
- Causing retain cycles through cell callbacks.

## Junior-Level Interview Answer

`UITableView` displays rows and `UICollectionView` displays flexible layouts like grids. Both use reusable cells. The data source provides cells and counts, while the delegate handles interactions and layout-related events.

## Senior-Level Interview Answer

I choose table view for straightforward lists and collection view for adaptive or mixed layouts. I prefer diffable data sources for state-driven updates, keep cell configuration idempotent, separate display models from domain models, cancel async work during reuse, and profile scrolling when cells become complex.

## Points To Remember

- Cells are temporary views, not data owners.
- Data source owns presentation state.
- Delegate owns interaction and some layout behavior.
- Reuse means every configure call must fully update visible state.
- Collection views are now the default for complex list and grid screens.

