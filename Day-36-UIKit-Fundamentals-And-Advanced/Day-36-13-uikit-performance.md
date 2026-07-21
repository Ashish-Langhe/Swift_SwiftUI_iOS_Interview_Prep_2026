# Day 36.13: UIKit Performance

UIKit performance is about preserving responsiveness. Most visible UIKit performance issues are dropped frames, slow screen transitions, delayed taps, laggy typing, and janky scrolling. A senior iOS engineer should know where the time goes and how to measure before changing code.

## Performance Mental Model

The main thread must stay free to:

- Process touch events.
- Run layout.
- Commit animations.
- Update visible cells.
- Render UI changes.

If you block the main thread, the app feels slow even when the code is technically correct.

## Smooth Scrolling

Common causes of poor scrolling:

- Synchronous image downloads.
- Image decoding on main thread.
- Complex Auto Layout in cells.
- Recreating formatters per cell.
- Large shadows without `shadowPath`.
- Too many nested stack views.
- Applying snapshots too often.

Bad:

```swift
func configure(with product: Product) {
    titleLabel.text = product.name
    priceLabel.text = NumberFormatter.currency.string(from: product.price)
    imageView.image = UIImage(data: try! Data(contentsOf: product.imageURL))
}
```

Better:

```swift
func configure(with row: ProductRow) {
    titleLabel.text = row.title
    priceLabel.text = row.price
    imageView.image = row.placeholder
}
```

Move formatting and image loading outside the cell.

## Cell Configuration Budget

Cells should:

- Assign already-prepared display values.
- Start cancellable image loading.
- Reset stale state.
- Avoid network, database, and expensive formatting.

Example:

```swift
final class ProductCell: UICollectionViewCell {
    private var representedID: Product.ID?

    func configure(with row: ProductRow, imageLoader: ImageLoading) {
        representedID = row.id
        titleLabel.text = row.title
        priceLabel.text = row.price
        imageView.image = UIImage(named: "placeholder")

        imageLoader.load(row.imageURL) { [weak self] image in
            guard self?.representedID == row.id else { return }
            self?.imageView.image = image
        }
    }

    override func prepareForReuse() {
        super.prepareForReuse()
        representedID = nil
        imageView.image = nil
    }
}
```

## Prefetching

```swift
extension FeedViewController: UICollectionViewDataSourcePrefetching {
    func collectionView(_ collectionView: UICollectionView, prefetchItemsAt indexPaths: [IndexPath]) {
        let urls = indexPaths.compactMap { dataSource.itemIdentifier(for: $0)?.imageURL }
        imageLoader.prefetch(urls)
    }

    func collectionView(_ collectionView: UICollectionView, cancelPrefetchingForItemsAt indexPaths: [IndexPath]) {
        let urls = indexPaths.compactMap { dataSource.itemIdentifier(for: $0)?.imageURL }
        imageLoader.cancelPrefetch(urls)
    }
}
```

Senior note: prefetching is a hint, not a guarantee. Code must still work when prefetch does not happen.

## Auto Layout Performance

Avoid constraint churn:

```swift
heightConstraint.constant = isExpanded ? 240 : 88
```

Instead of:

```swift
removeConstraints(oldConstraints)
addConstraints(newConstraints)
```

Use:

- Constraint references.
- Reasonable view hierarchy depth.
- Estimated cell heights.
- Simple constraints in heavily reused cells.

## Diffable Snapshot Performance

Snapshot application has a cost. Avoid applying for every tiny intermediate state.

Bad:

```swift
func searchTextDidChange(_ text: String) {
    applySnapshot(filteredRows(text))
}
```

Better:

```swift
func searchTextDidChange(_ text: String) {
    searchDebouncer.schedule { [weak self] in
        self?.applySnapshot(self?.filteredRows(text) ?? [])
    }
}
```

Senior thinking:

- Debounce search.
- Coalesce state changes.
- Disable animation for huge updates.
- Use reload-style apply when replacing everything.

## Main Thread Work

Expensive work that should usually not run on the main thread:

- JSON decoding for large payloads.
- Image decoding and resizing.
- Database queries.
- Cryptography.
- Large sorting/filtering.
- File IO.

Example:

```swift
Task {
    let rows = await mapper.makeRows(from: products)

    await MainActor.run {
        self.applySnapshot(rows)
    }
}
```

## Measuring With Instruments

Use:

- Time Profiler for CPU hotspots.
- Allocations for memory churn.
- Core Animation for frame drops and rendering issues.
- Main Thread Checker for UI thread misuse.
- Memory Graph for retain cycles.

Senior workflow:

1. Reproduce the performance problem.
2. Record a trace.
3. Find the biggest cost.
4. Make one change.
5. Re-measure.

## Image Performance

Large images hurt:

- Memory footprint.
- Decode time.
- Scrolling performance.
- Cache pressure.

Good image loader behavior:

- Downsample to target size.
- Decode off main thread.
- Cache memory and disk wisely.
- Cancel requests for reused cells.
- Avoid duplicate in-flight requests.

## Layout Thrashing

Layout thrashing happens when code repeatedly changes layout and forces layout passes.

Bad:

```swift
for view in views {
    view.isHidden.toggle()
    parent.layoutIfNeeded()
}
```

Better:

```swift
for view in views {
    view.isHidden.toggle()
}
parent.setNeedsLayout()
```

Only call `layoutIfNeeded` inside animations or when you truly need immediate layout.

## Memory And Performance

Performance issues can be memory issues:

- Too many cached images.
- Retained view controllers.
- Large snapshots.
- Cell closures retaining controllers.
- Timers/tasks keeping screens alive.

Use deinit logs during investigation:

```swift
deinit {
    print("FeedViewController deinit")
}
```

Then confirm with Memory Graph.

## Common Mistakes

- Optimizing without measuring.
- Treating `reloadData` as harmless.
- Formatting dates in cells.
- Loading images synchronously.
- Forgetting cancellation.
- Adding shadows to every cell without measuring.
- Applying animated diffs to massive updates.

## Junior-Level Interview Answer

UIKit performance means keeping scrolling and interactions smooth. I avoid doing heavy work on the main thread, reuse cells properly, load images asynchronously, and use Instruments to find slow code.

## Senior-Level Interview Answer

I treat performance as a measured feedback loop. I keep cell configuration cheap, move transformation and IO off the main thread, use cancellable image loading and prefetching, control diffable snapshot frequency, avoid layout churn, and verify with Time Profiler, Core Animation, Allocations, and Memory Graph. I optimize the biggest measured cost first.

## Points To Remember

- Main thread responsiveness is the core budget.
- Cells should render, not compute.
- Image downsampling matters.
- Snapshot updates are not free.
- Constraint churn hurts reused views.
- Measure before and after each optimization.

