# Day 25: Layout Basics

## SwiftUI Layout Mental Model

SwiftUI layout is a negotiation:

1. Parent proposes a size.
2. Child chooses a size.
3. Parent positions the child.

This is different from Auto Layout constraints. You describe relationships and preferences, then SwiftUI computes placement.

## Core Containers

Use stacks for simple layout:

```swift
VStack(alignment: .leading, spacing: 12) {
    Text("Order #1234")
        .font(.headline)

    Text("Arrives tomorrow")
        .foregroundStyle(.secondary)
}
```

Horizontal:

```swift
HStack(spacing: 12) {
    Image(systemName: "shippingbox")
    Text("Preparing")
    Spacer()
    Text("2 items")
}
```

Overlay:

```swift
ZStack(alignment: .topTrailing) {
    ProductImage(url: product.imageURL)
    DiscountBadge(percent: product.discountPercent)
}
```

## Spacer

`Spacer` consumes flexible space.

```swift
HStack {
    Text(product.name)
    Spacer()
    Text(product.priceText)
}
```

Use it when the layout should push content apart.

## Frame

`frame` changes the size a view asks for or accepts.

```swift
Text("Checkout")
    .frame(maxWidth: .infinity)
```

This makes the text view expand horizontally within its parent.

Button example:

```swift
Button("Pay Now") {
    pay()
}
.buttonStyle(.borderedProminent)
.frame(maxWidth: .infinity)
```

## Padding

Padding adds space around content.

```swift
Text("Limited offer")
    .padding(.horizontal, 12)
    .padding(.vertical, 8)
```

Order matters in SwiftUI modifiers.

```swift
Text("Sale")
    .padding()
    .background(.red)
```

This backgrounds the padded area.

```swift
Text("Sale")
    .background(.red)
    .padding()
```

This backgrounds only the text area, then adds outer spacing.

## Alignment

Use alignment to control how children line up.

```swift
VStack(alignment: .leading, spacing: 8) {
    Text("Profile")
        .font(.headline)
    Text("Senior iOS Engineer")
        .foregroundStyle(.secondary)
}
```

A common beginner mistake is using too many spacers when alignment would be clearer.

## Lazy Containers

Use lazy stacks and grids for large scrolling content.

```swift
ScrollView {
    LazyVStack(spacing: 12) {
        ForEach(products) { product in
            ProductRow(product: product)
        }
    }
    .padding()
}
```

Lazy containers create views as needed, which improves performance for large lists.

## Grid Example

```swift
let columns = [
    GridItem(.adaptive(minimum: 160), spacing: 16)
]

ScrollView {
    LazyVGrid(columns: columns, spacing: 16) {
        ForEach(products) { product in
            ProductCard(product: product)
        }
    }
    .padding()
}
```

This works well for product galleries, photo grids, and dashboard cards.

## Safe Area

Respect safe areas by default.

```swift
VStack {
    content
}
.safeAreaInset(edge: .bottom) {
    CheckoutBar(total: total)
}
```

Use `ignoresSafeArea` carefully for backgrounds or full-screen media, not ordinary controls.

## Dynamic Type and Accessibility

Layout should survive larger text.

```swift
Text(product.description)
    .font(.body)
    .fixedSize(horizontal: false, vertical: true)
```

Senior engineers test:

- Large dynamic type
- Long localized strings
- Right-to-left layouts
- Small devices
- Split view or iPad multitasking

## Real iOS Example: Product Row

```swift
struct ProductRow: View {
    let product: Product

    var body: some View {
        HStack(alignment: .top, spacing: 12) {
            AsyncImage(url: product.imageURL) { image in
                image
                    .resizable()
                    .scaledToFill()
            } placeholder: {
                Color.secondary.opacity(0.15)
            }
            .frame(width: 64, height: 64)
            .clipShape(RoundedRectangle(cornerRadius: 8))

            VStack(alignment: .leading, spacing: 6) {
                Text(product.name)
                    .font(.headline)

                Text(product.subtitle)
                    .font(.subheadline)
                    .foregroundStyle(.secondary)

                Text(product.priceText)
                    .font(.callout.weight(.semibold))
            }

            Spacer(minLength: 8)
        }
        .padding(.vertical, 8)
    }
}
```

This layout is simple, resilient, and readable.

## Latest SwiftUI Layout Notes

Recent SwiftUI releases continue expanding interaction and container capabilities, including reorderable containers beyond `List`, broader swipe-action support, and toolbar behavior improvements. These features are layout-adjacent because they require stable containers, identity, and adaptable content.

## Senior iOS Engineer Perspective

Senior layout thinking:

- Start with content hierarchy.
- Prefer simple stacks before custom layout.
- Keep modifier order intentional.
- Design for dynamic type and localization.
- Avoid hard-coded sizes except for genuinely fixed assets.
- Use lazy containers for large scrolling data.
- Profile if layout becomes expensive.

## Interview Notes

Junior:

Use `VStack`, `HStack`, `ZStack`, `Spacer`, `frame`, and `padding` to build layouts.

Mid-level:

SwiftUI layout is parent proposal, child choice, parent placement. Modifier order matters.

Senior:

I design layout around content behavior, accessibility, localization, identity, and performance. I avoid brittle fixed-size layouts and test with real data, long text, and large dynamic type.

## Practice

1. Build a product row using `HStack` and `VStack`.
2. Convert a vertical list to `LazyVStack`.
3. Create an adaptive grid.
4. Explain how modifier order changes layout.
