# Day 36.14: Accessibility, Dynamic Type, And Localization In UIKit

Accessibility, Dynamic Type, and localization are not optional polish for UIKit apps. They are part of production quality. Senior iOS engineers are expected to design UI that works for VoiceOver users, large text sizes, right-to-left languages, different regions, and different input modes.

## Accessibility Mental Model

UIKit exposes UI to assistive technologies through accessibility elements.

Important properties:

- `isAccessibilityElement`
- `accessibilityLabel`
- `accessibilityValue`
- `accessibilityHint`
- `accessibilityTraits`
- `accessibilityIdentifier`

Do not confuse `accessibilityLabel` with `accessibilityIdentifier`.

- Label is for users.
- Identifier is for UI tests.

## Basic Accessibility

```swift
favoriteButton.accessibilityLabel = "Add to favorites"
favoriteButton.accessibilityHint = "Adds this product to your saved items"
favoriteButton.accessibilityTraits = [.button]
favoriteButton.accessibilityIdentifier = "product.favorite.button"
```

For images:

```swift
productImageView.isAccessibilityElement = true
productImageView.accessibilityLabel = "Photo of \(product.name)"
```

Decorative images:

```swift
separatorImageView.isAccessibilityElement = false
```

## Grouping Custom Views

For a tappable card:

```swift
cardView.isAccessibilityElement = true
cardView.accessibilityLabel = "\(product.name), \(product.formattedPrice)"
cardView.accessibilityHint = "Opens product details"
cardView.accessibilityTraits = [.button]
```

This prevents VoiceOver from reading every internal label separately when the card behaves as one item.

## Accessibility Custom Actions

```swift
cell.accessibilityCustomActions = [
    UIAccessibilityCustomAction(
        name: "Add to cart",
        target: self,
        selector: #selector(addToCartAction)
    )
]

@objc private func addToCartAction() -> Bool {
    onAddToCart?()
    return true
}
```

Use custom actions for swipe-like or secondary actions that should be available to VoiceOver users.

## Dynamic Type

Use preferred fonts:

```swift
titleLabel.font = UIFont.preferredFont(forTextStyle: .headline)
titleLabel.adjustsFontForContentSizeCategory = true
titleLabel.numberOfLines = 0
```

For custom fonts:

```swift
let baseFont = UIFont(name: "AvenirNext-DemiBold", size: 17)!
titleLabel.font = UIFontMetrics(forTextStyle: .headline).scaledFont(for: baseFont)
titleLabel.adjustsFontForContentSizeCategory = true
```

Senior checklist:

- No fixed label heights.
- Buttons can grow vertically.
- Stacks can wrap or reflow.
- Table/collection cells self-size.
- Test accessibility text sizes, not just normal large.

## Dynamic Type Layout Example

Bad:

```swift
titleLabel.heightAnchor.constraint(equalToConstant: 20).isActive = true
```

Better:

```swift
titleLabel.numberOfLines = 0
titleLabel.setContentCompressionResistancePriority(.required, for: .vertical)
```

Let content determine height.

## VoiceOver Order

Sometimes visual order and accessibility order differ.

```swift
view.accessibilityElements = [
    titleLabel,
    priceLabel,
    addToCartButton
]
```

Use this carefully when custom layout would otherwise create a confusing reading order.

## Announcements

When important UI state changes:

```swift
UIAccessibility.post(
    notification: .announcement,
    argument: "Item added to cart"
)
```

For screen changes:

```swift
UIAccessibility.post(
    notification: .screenChanged,
    argument: titleLabel
)
```

## Localization Basics

Use localized strings:

```swift
titleLabel.text = String(localized: "orders.title")
```

Avoid concatenating localized text:

```swift
// Bad
label.text = "Hello " + user.name
```

Better:

```swift
label.text = String(localized: "profile.greeting \(user.name)")
```

Localization affects:

- Text length.
- Date format.
- Currency format.
- Number format.
- Plurals.
- Right-to-left layout.

## Right-To-Left Support

Use leading/trailing:

```swift
titleLabel.leadingAnchor.constraint(equalTo: contentView.leadingAnchor, constant: 16)
chevron.trailingAnchor.constraint(equalTo: contentView.trailingAnchor, constant: -16)
```

Avoid left/right unless physical direction is truly intended.

For images:

```swift
imageView.image = UIImage(systemName: "chevron.forward")
```

Prefer semantic SF Symbols like `forward` when possible.

## Accessibility And Collection Cells

```swift
func configure(with row: OrderRow) {
    var content = UIListContentConfiguration.subtitleCell()
    content.text = row.title
    content.secondaryText = row.status
    self.contentConfiguration = content

    accessibilityLabel = "\(row.title), \(row.status)"
    accessibilityHint = "Opens order details"
    accessibilityTraits = [.button]
}
```

Do not leave stale accessibility labels during reuse.

## Common Mistakes

- Fixed heights with large text.
- Meaningful icons without labels.
- Decorative images read by VoiceOver.
- UI test identifiers used as user labels.
- Concatenated localized strings.
- Left/right constraints breaking RTL.
- Custom controls without traits.
- Missing accessibility for custom actions.

## Junior-Level Interview Answer

Accessibility helps users with VoiceOver and other assistive technologies. Dynamic Type lets text follow the user's preferred size. Localization adapts text, dates, numbers, and layout for different languages and regions.

## Senior-Level Interview Answer

I design UIKit screens for accessibility from the start. I use semantic labels, traits, values, hints, custom actions, VoiceOver order, and Dynamic Type friendly constraints. For localization, I avoid string concatenation, use leading/trailing constraints, test longer languages and RTL, and ensure cells reset accessibility state during reuse.

## Points To Remember

- Accessibility label is user-facing.
- Accessibility identifier is for tests.
- Dynamic Type requires flexible vertical layout.
- Use leading/trailing for RTL.
- Decorative images should be hidden from accessibility.
- Custom controls need accessibility traits and actions.

