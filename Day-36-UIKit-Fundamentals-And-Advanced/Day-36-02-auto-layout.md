# Day 36.2: Auto Layout

Auto Layout is UIKit's constraint-based layout system. A junior engineer should know how to pin views and avoid ambiguous layouts. A senior engineer should understand constraint priority, intrinsic content size, compression resistance, hugging, safe areas, scroll views, self-sizing cells, layout debugging, and performance.

## What Auto Layout Solves

Auto Layout lets you describe relationships instead of hard-coded frames.

Examples:

- Button is 16 points from the trailing safe area.
- Label is centered vertically with image.
- Card width follows readable content guide.
- Text field grows with Dynamic Type.
- Cell height is based on content.

Frame-based layout says "put this at x/y/width/height." Auto Layout says "these relationships must be true."

## Basic Programmatic Setup

```swift
final class LoginView: UIView {
    private let emailField = UITextField()
    private let passwordField = UITextField()
    private let loginButton = UIButton(type: .system)

    override init(frame: CGRect) {
        super.init(frame: frame)
        backgroundColor = .systemBackground
        setupViews()
        setupConstraints()
    }

    required init?(coder: NSCoder) {
        fatalError("init(coder:) has not been implemented")
    }

    private func setupViews() {
        emailField.placeholder = "Email"
        passwordField.placeholder = "Password"
        passwordField.isSecureTextEntry = true
        loginButton.setTitle("Sign In", for: .normal)

        [emailField, passwordField, loginButton].forEach {
            $0.translatesAutoresizingMaskIntoConstraints = false
            addSubview($0)
        }
    }

    private func setupConstraints() {
        NSLayoutConstraint.activate([
            emailField.topAnchor.constraint(equalTo: safeAreaLayoutGuide.topAnchor, constant: 32),
            emailField.leadingAnchor.constraint(equalTo: layoutMarginsGuide.leadingAnchor),
            emailField.trailingAnchor.constraint(equalTo: layoutMarginsGuide.trailingAnchor),

            passwordField.topAnchor.constraint(equalTo: emailField.bottomAnchor, constant: 12),
            passwordField.leadingAnchor.constraint(equalTo: emailField.leadingAnchor),
            passwordField.trailingAnchor.constraint(equalTo: emailField.trailingAnchor),

            loginButton.topAnchor.constraint(equalTo: passwordField.bottomAnchor, constant: 20),
            loginButton.centerXAnchor.constraint(equalTo: centerXAnchor)
        ])
    }
}
```

Key rule: set `translatesAutoresizingMaskIntoConstraints = false` for views you constrain manually.

## Anchors

UIKit provides typed anchors:

- `topAnchor`, `bottomAnchor`
- `leadingAnchor`, `trailingAnchor`
- `centerXAnchor`, `centerYAnchor`
- `widthAnchor`, `heightAnchor`
- `firstBaselineAnchor`, `lastBaselineAnchor`

Use leading/trailing instead of left/right for localization.

## Safe Area And Margins

Use safe areas for screen edges that interact with status bars, home indicator, Dynamic Island, toolbars, and navigation bars.

```swift
NSLayoutConstraint.activate([
    tableView.topAnchor.constraint(equalTo: view.safeAreaLayoutGuide.topAnchor),
    tableView.leadingAnchor.constraint(equalTo: view.leadingAnchor),
    tableView.trailingAnchor.constraint(equalTo: view.trailingAnchor),
    tableView.bottomAnchor.constraint(equalTo: view.bottomAnchor)
])
```

Use layout margins for readable content:

```swift
titleLabel.leadingAnchor.constraint(equalTo: view.layoutMarginsGuide.leadingAnchor)
```

Senior thinking:

- Full-bleed media often pins to `view`.
- Controls and text often pin to `safeAreaLayoutGuide` or `layoutMarginsGuide`.
- iPad layouts often benefit from `readableContentGuide`.

## Intrinsic Content Size

Some views know their natural size:

- `UILabel`
- `UIButton`
- `UIImageView` with image
- `UITextField`

Example:

```swift
let label = UILabel()
label.numberOfLines = 0
```

If the label has leading, trailing, and top constraints, Auto Layout can calculate height from text.

## Content Hugging And Compression Resistance

Content hugging says: "Do not grow me more than needed."

Compression resistance says: "Do not shrink me smaller than content."

Example:

```swift
nameLabel.setContentCompressionResistancePriority(.defaultLow, for: .horizontal)
priceLabel.setContentCompressionResistancePriority(.required, for: .horizontal)
```

Use case: In a product row, the price should remain visible while the product name truncates.

## Priority

Constraint priorities let the layout bend instead of break.

```swift
let maxWidth = cardView.widthAnchor.constraint(lessThanOrEqualToConstant: 420)
maxWidth.priority = .required

let centered = cardView.centerXAnchor.constraint(equalTo: centerXAnchor)
centered.priority = .defaultHigh

NSLayoutConstraint.activate([maxWidth, centered])
```

Common priorities:

- `1000`: required
- `750`: default high
- `250`: default low

Senior tip: prefer a soft constraint over fighting runtime unsatisfiable-constraint warnings.

## UIStackView

`UIStackView` is not a drawing view. It manages arranged subviews.

```swift
let stack = UIStackView(arrangedSubviews: [titleLabel, subtitleLabel, button])
stack.axis = .vertical
stack.spacing = 8
stack.alignment = .fill
stack.distribution = .fill
```

Good for:

- Forms
- Vertical content
- Button rows
- Empty states
- Settings screens

Avoid deeply nested stack views in very large scrolling cells if performance becomes an issue.

## Scroll View Auto Layout

Scroll views have two important guides:

- `contentLayoutGuide`: content size
- `frameLayoutGuide`: visible frame

Example:

```swift
let scrollView = UIScrollView()
let contentView = UIView()

scrollView.addSubview(contentView)
contentView.translatesAutoresizingMaskIntoConstraints = false

NSLayoutConstraint.activate([
    contentView.topAnchor.constraint(equalTo: scrollView.contentLayoutGuide.topAnchor),
    contentView.leadingAnchor.constraint(equalTo: scrollView.contentLayoutGuide.leadingAnchor),
    contentView.trailingAnchor.constraint(equalTo: scrollView.contentLayoutGuide.trailingAnchor),
    contentView.bottomAnchor.constraint(equalTo: scrollView.contentLayoutGuide.bottomAnchor),

    contentView.widthAnchor.constraint(equalTo: scrollView.frameLayoutGuide.widthAnchor)
])
```

The width constraint is what makes vertical scrolling work predictably.

## Self-Sizing Cells

Use constraints that fully describe cell content from top to bottom.

```swift
final class MessageCell: UITableViewCell {
    private let messageLabel = UILabel()

    override init(style: UITableViewCell.CellStyle, reuseIdentifier: String?) {
        super.init(style: style, reuseIdentifier: reuseIdentifier)
        messageLabel.numberOfLines = 0
        messageLabel.translatesAutoresizingMaskIntoConstraints = false
        contentView.addSubview(messageLabel)

        NSLayoutConstraint.activate([
            messageLabel.topAnchor.constraint(equalTo: contentView.topAnchor, constant: 12),
            messageLabel.leadingAnchor.constraint(equalTo: contentView.leadingAnchor, constant: 16),
            messageLabel.trailingAnchor.constraint(equalTo: contentView.trailingAnchor, constant: -16),
            messageLabel.bottomAnchor.constraint(equalTo: contentView.bottomAnchor, constant: -12)
        ])
    }

    required init?(coder: NSCoder) {
        fatalError("init(coder:) has not been implemented")
    }
}
```

For tables:

```swift
tableView.rowHeight = UITableView.automaticDimension
tableView.estimatedRowHeight = 80
```

## Debugging Auto Layout

Common symptoms:

- "Unable to simultaneously satisfy constraints"
- Ambiguous layout
- Unexpected zero height
- Jumping cell heights
- Clipped text with Dynamic Type

Useful tactics:

```swift
someConstraint.identifier = "Avatar width"
```

Use:

- Xcode View Debugger
- Constraint identifiers
- `UIViewAlertForUnsatisfiableConstraints` symbolic breakpoint
- Dynamic Type testing
- Right-to-left language testing

## Performance

Auto Layout is fast enough for normal UI, but can become expensive with:

- Too many nested stack views in scrolling cells.
- Recreating constraints repeatedly.
- Calling `layoutIfNeeded` inside tight loops.
- Complex self-sizing cells without estimates.

Better approach:

```swift
private var expandedConstraint: NSLayoutConstraint!

func setExpanded(_ isExpanded: Bool) {
    expandedConstraint.constant = isExpanded ? 240 : 80
    UIView.animate(withDuration: 0.25) {
        self.layoutIfNeeded()
    }
}
```

Keep references and update constants instead of destroying and recreating constraints.

## Common Interview Traps

- `translatesAutoresizingMaskIntoConstraints` left as `true`.
- Missing bottom constraint in self-sizing cells.
- Pinning text to unsafe screen edges.
- Using left/right instead of leading/trailing.
- Confusing content hugging with compression resistance.
- Adding constraints every time layout runs.

## Junior-Level Interview Answer

Auto Layout uses constraints to position and size views for different devices and content sizes. I create constraints with anchors, use safe areas, disable autoresizing mask translation, and make sure views have enough constraints to determine x, y, width, and height.

## Senior-Level Interview Answer

Auto Layout is a constraint-solving system. I design layouts around intrinsic size, priorities, safe areas, margins, Dynamic Type, and scroll-view guides. For performance, I avoid constraint churn in cells, use good estimated sizes, identify constraints for debugging, and choose stack views or manual constraints based on complexity and scrolling cost.

## Points To Remember

- Constraints describe relationships.
- Safe area protects interactive screen regions.
- `contentLayoutGuide` defines scroll content.
- `frameLayoutGuide` defines scroll viewport.
- Hugging controls growth.
- Compression resistance controls shrinking.
- Constraint warnings are bugs, even if UI "looks fine."

