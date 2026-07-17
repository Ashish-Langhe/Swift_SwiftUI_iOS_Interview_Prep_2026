# Day 26: Accessibility, Dynamic Type, And Localization

## Why This Topic Matters

SwiftUI makes accessible UI easier, but it does not make it automatic. Senior iOS engineers must think about:

- VoiceOver
- Dynamic Type
- Color contrast
- Reduce Motion
- Hit targets
- Localization
- Right-to-left layout
- Long text
- Semantic controls

This is not optional polish. It is product quality.

## Prefer Semantic Controls

Good:

```swift
Button("Delete", role: .destructive) {
    delete()
}
```

Weak:

```swift
Text("Delete")
    .onTapGesture {
        delete()
    }
```

Semantic controls carry accessibility behavior, focus, roles, and platform interaction.

## Accessibility Labels

```swift
Image(systemName: "heart.fill")
    .foregroundStyle(.pink)
    .accessibilityLabel("Favorite")
```

For decorative images:

```swift
Image("background-pattern")
    .accessibilityHidden(true)
```

Do not make VoiceOver read meaningless decoration.

## Combine Children

```swift
HStack {
    Text(product.name)
    Text(product.priceText)
}
.accessibilityElement(children: .combine)
```

This can make row reading smoother.

## Custom Actions

```swift
ProductRow(product: product)
    .accessibilityAction(named: "Favorite") {
        favorite(product)
    }
    .accessibilityAction(named: "Delete") {
        delete(product)
    }
```

Useful when visual UI has swipe/context actions.

## Dynamic Type

Design for larger text.

```swift
Text(product.description)
    .font(.body)
    .fixedSize(horizontal: false, vertical: true)
```

Avoid locking text into tiny fixed frames.

Bad:

```swift
Text(title)
    .frame(height: 20)
```

That will clip with larger text.

## Hit Targets

Small icons need tappable area.

```swift
Button {
    favorite()
} label: {
    Image(systemName: "heart")
        .frame(width: 44, height: 44)
}
```

Visual size and hit target size are not always the same.

## Reduce Motion

```swift
@Environment(\.accessibilityReduceMotion) private var reduceMotion

content
    .animation(reduceMotion ? nil : .spring, value: isExpanded)
```

Do not force motion-heavy transitions on users who asked to reduce motion.

## Localization

Use localized strings and avoid manual string assembly where grammar may change.

Weak:

```swift
Text("\(count) items")
```

Better:

```swift
Text("cart.item.count \(count)")
```

In real projects, use String Catalogs and pluralization rules.

## Right-To-Left Layout

Avoid hard-coding left/right when leading/trailing expresses intent.

```swift
VStack(alignment: .leading) {
    Text(title)
}
.padding(.leading)
```

Use `leading` and `trailing` instead of `left` and `right` for user-facing layout.

## Common Mistakes

- Gesture-only actions.
- Missing labels for icon-only buttons.
- Fixed-height text containers.
- Poor contrast.
- Ignoring reduce motion.
- Using left/right instead of leading/trailing.
- Testing only English short strings.
- Hiding important actions in swipe-only UI.

## Senior iOS Engineer Perspective

Accessibility and localization are architecture concerns:

- Are actions semantic?
- Does every icon-only control have a label?
- Does layout survive large text?
- Can VoiceOver users perform row actions?
- Are strings localizable with pluralization?
- Does layout work in RTL?
- Are animations respectful of user settings?

## Interview Notes

Junior:

SwiftUI supports accessibility labels and dynamic type.

Mid-level:

Use semantic controls, labels, custom actions, large text previews, and localized strings.

Senior:

I treat accessibility, dynamic type, and localization as required acceptance criteria. I test with assistive settings, avoid gesture-only actions, and design layout and strings for real-world variation.

## Practice

1. Add labels to icon-only buttons.
2. Add large dynamic type previews.
3. Add accessibility custom actions to a row.
4. Replace left/right layout with leading/trailing.
