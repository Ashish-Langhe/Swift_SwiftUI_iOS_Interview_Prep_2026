# Day 36.8: UIKit Advanced Patterns

UIKit at senior level is less about knowing every class and more about building predictable, testable, accessible, and performant screens. This file collects the advanced decisions that often separate a working UIKit screen from production-quality UIKit engineering.

## Programmatic UI Structure

A clean UIKit screen often has:

- Custom root view.
- View controller for lifecycle and binding.
- View model or presenter for state.
- Coordinator for navigation.
- Services injected through protocols.

Example:

```swift
final class ProductListViewController: UIViewController {
    private let productView = ProductListView()
    private let viewModel: ProductListViewModel
    private let coordinator: ProductListCoordinating

    init(viewModel: ProductListViewModel, coordinator: ProductListCoordinating) {
        self.viewModel = viewModel
        self.coordinator = coordinator
        super.init(nibName: nil, bundle: nil)
    }

    required init?(coder: NSCoder) {
        fatalError("init(coder:) has not been implemented")
    }

    override func loadView() {
        view = productView
    }

    override func viewDidLoad() {
        super.viewDidLoad()
        bind()
        viewModel.load()
    }
}
```

Why this helps:

- Root view can be tested visually and reused.
- Controller does less layout work.
- View model can be unit tested.
- Coordinator keeps navigation out of cells and views.

## View State Rendering

Use explicit state:

```swift
enum ProductListState {
    case idle
    case loading
    case loaded([ProductRow])
    case empty
    case failed(message: String)
}

func render(_ state: ProductListState) {
    switch state {
    case .idle:
        contentView.isHidden = true
        loadingView.stopAnimating()
    case .loading:
        loadingView.startAnimating()
        errorView.isHidden = true
    case .loaded(let rows):
        loadingView.stopAnimating()
        applySnapshot(rows)
    case .empty:
        showEmptyState()
    case .failed(let message):
        showError(message)
    }
}
```

Senior thinking: render from state rather than scattering boolean flags across callbacks.

## UIActions And Modern Buttons

Modern UIKit supports action-based controls.

```swift
let retryButton = UIButton(
    configuration: .filled(),
    primaryAction: UIAction(title: "Retry") { [weak self] _ in
        self?.viewModel.retry()
    }
)
```

Benefits:

- Less selector boilerplate.
- Action lives near configuration.
- Good for small buttons.

Use selectors when:

- Objective-C interoperability matters.
- Target-action is shared across many controls.
- Existing codebase uses selector-heavy UIKit.

## UIButton.Configuration

```swift
var configuration = UIButton.Configuration.filled()
configuration.title = "Pay Now"
configuration.image = UIImage(systemName: "creditcard")
configuration.imagePadding = 8
configuration.cornerStyle = .medium

payButton.configuration = configuration
```

Senior note: prefer configurations for modern UIKit buttons instead of manually tuning many title/image edge insets.

## UIContentConfiguration

For reusable content:

```swift
struct BadgeContentConfiguration: UIContentConfiguration {
    let title: String
    let color: UIColor

    func makeContentView() -> UIView & UIContentView {
        BadgeContentView(configuration: self)
    }

    func updated(for state: UIConfigurationState) -> BadgeContentConfiguration {
        self
    }
}
```

This pattern is useful for custom cells because configuration becomes a value object.

## Trait Collections

UIKit screens must respond to environment changes:

- Dark mode.
- Dynamic Type.
- Size class.
- Display scale.
- Accessibility contrast.
- iPad multitasking.

Example:

```swift
override func traitCollectionDidChange(_ previousTraitCollection: UITraitCollection?) {
    super.traitCollectionDidChange(previousTraitCollection)

    if traitCollection.hasDifferentColorAppearance(comparedTo: previousTraitCollection) {
        updateColors()
    }
}
```

## Dynamic Type

```swift
titleLabel.font = UIFont.preferredFont(forTextStyle: .headline)
titleLabel.adjustsFontForContentSizeCategory = true
titleLabel.numberOfLines = 0
```

Senior checklist:

- Test largest accessibility sizes.
- Avoid fixed-height labels.
- Avoid clipping text in buttons.
- Use `UIFontMetrics` for custom fonts.

## Accessibility

```swift
favoriteButton.accessibilityLabel = "Add to favorites"
favoriteButton.accessibilityHint = "Adds this product to your saved items"
favoriteButton.accessibilityTraits = [.button]
```

For custom views:

```swift
cardView.isAccessibilityElement = true
cardView.accessibilityLabel = "\(product.name), \(product.price)"
cardView.accessibilityTraits = [.button]
```

Senior thinking: accessibility is part of quality, not a final polish task.

## Keyboard Handling

Use keyboard layout guide where possible.

```swift
NSLayoutConstraint.activate([
    formStack.bottomAnchor.constraint(
        lessThanOrEqualTo: view.keyboardLayoutGuide.topAnchor,
        constant: -16
    )
])
```

This avoids manual keyboard notification math for many layouts.

## Modern Sheet Presentation

```swift
let controller = FilterViewController()
if let sheet = controller.sheetPresentationController {
    sheet.detents = [.medium(), .large()]
    sheet.prefersGrabberVisible = true
}
present(controller, animated: true)
```

Use cases:

- Filters.
- Checkout summary.
- Sharing options.
- Account actions.

## Architecture Boundaries

Avoid:

```swift
cell.onTapBuy = {
    navigationController?.pushViewController(...)
}
```

Better:

```swift
cell.onTapBuy = { [weak self] productID in
    self?.coordinator.showCheckout(productID: productID)
}
```

Cells should not navigate. Views should not know services. Controllers should not parse API responses.

## Testing UIKit

You can unit test:

- View models.
- Snapshot builders.
- Coordinators with fake navigation.
- Formatters.
- Cell configuration values.

Example:

```swift
func testLoadedStateBuildsProductRows() {
    let products = [Product(id: UUID(), name: "Keyboard", price: 129)]
    let rows = ProductListMapper().map(products)
    XCTAssertEqual(rows.first?.title, "Keyboard")
}
```

For UI:

- Use accessibility identifiers.
- Test critical flows with XCUITest.
- Use snapshot tests if your team supports them.

## Common Mistakes

- Massive view controllers.
- Hard-coded frames that fail on iPad or Dynamic Type.
- Navigation from cells.
- Business logic in views.
- Missing accessibility.
- Strong callback cycles.
- Ignoring trait changes.
- Reloading entire lists for tiny changes.

## Junior-Level Interview Answer

Advanced UIKit means organizing screens with clear responsibilities, using Auto Layout correctly, handling accessibility and Dynamic Type, and using modern APIs like diffable data source, compositional layout, and button configurations.

## Senior-Level Interview Answer

For production UIKit, I think in boundaries: view, controller, state, navigation, and services. I render from explicit state, keep cells idempotent, use modern configuration APIs, make layouts adaptive, test core mapping and navigation decisions, and profile before optimizing. UIKit gives control, but that control must be disciplined.

## Points To Remember

- Programmatic UIKit benefits from custom root views.
- Render explicit state.
- Keep navigation in coordinators.
- Use modern configuration APIs.
- Accessibility and Dynamic Type are baseline requirements.
- Senior UIKit is architecture plus lifecycle plus performance.

