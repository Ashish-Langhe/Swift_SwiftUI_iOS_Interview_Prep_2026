# Day 26: Previews And Preview-Driven Development

## Why Previews Matter

SwiftUI previews are not just visual snapshots. Used well, they help you design, debug, and review states quickly.

Good previews cover:

- Loading
- Empty
- Error
- Loaded
- Long text
- Dark mode
- Large dynamic type
- Different locales
- Permission denied
- Offline

## Basic Preview

```swift
#Preview {
    ProductRow(product: .sample)
        .padding()
}
```

Keep sample data close enough to reality that layout issues appear early.

## Preview Multiple States

```swift
#Preview("Loaded") {
    ProductsScreen(model: .previewLoaded)
}

#Preview("Empty") {
    ProductsScreen(model: .previewEmpty)
}

#Preview("Error") {
    ProductsScreen(model: .previewError)
}
```

Senior tip: if a screen has a state enum, it should usually have previews for each important state.

## Preview Data Factories

```swift
extension Product {
    static let sample = Product(
        id: UUID(uuidString: "11111111-1111-1111-1111-111111111111")!,
        name: "Wireless Keyboard",
        price: 129
    )

    static let longName = Product(
        id: UUID(uuidString: "22222222-2222-2222-2222-222222222222")!,
        name: "Ultra Compact Backlit Wireless Mechanical Keyboard For Multi-Device Workflows",
        price: 149
    )
}
```

Use stable sample IDs to avoid preview identity noise.

## Environment Previews

```swift
#Preview("Large Text") {
    ProductRow(product: .longName)
        .environment(\.dynamicTypeSize, .accessibility3)
}

#Preview("Dark") {
    ProductRow(product: .sample)
        .preferredColorScheme(.dark)
}
```

This catches real production issues before manual QA.

## Preview with Stub Services

```swift
struct PreviewProductService: ProductService {
    func products() async throws -> [Product] {
        [.sample, .longName]
    }
}

#Preview {
    ProductsScreen(
        model: ProductsModel(service: PreviewProductService())
    )
}
```

Never rely on live network calls in previews.

## Preview Matrix

For important reusable components:

```swift
#Preview("Primary Button States") {
    VStack(spacing: 16) {
        PrimaryButton(title: "Continue", isLoading: false) {}
        PrimaryButton(title: "Saving", isLoading: true) {}
        PrimaryButton(title: "Very Long Localized Button Title", isLoading: false) {}
    }
    .padding()
}
```

Preview matrices are especially useful for design-system components.

## What Previews Are Not

Previews are not a replacement for:

- Unit tests
- UI tests
- Accessibility audits
- Device testing
- Performance profiling

They are fast feedback, not proof of correctness.

## Common Mistakes

- Only previewing happy path.
- Using live services.
- Sample data that is too short and clean.
- No large dynamic type previews.
- No dark mode previews.
- Preview code that is harder to create than the screen itself.
- Letting previews rot after refactors.

## Senior iOS Engineer Perspective

Senior preview design asks:

- What states can this screen enter?
- What data breaks layout?
- What dependencies need stubs?
- Can a reviewer quickly inspect behavior?
- Can design/product see important variants?
- Are previews part of the component API discipline?

## Interview Notes

Junior:

Previews show SwiftUI views without running the full app.

Mid-level:

Use previews for multiple states, dark mode, dynamic type, and stub data.

Senior:

I use previews as fast state documentation and design review tooling. Every important state and reusable component should be cheap to instantiate with realistic data.

## Practice

1. Add loaded, empty, and error previews for a screen.
2. Add dark mode and large text previews for a row.
3. Replace live preview networking with a stub service.
4. Build a preview matrix for a custom button.
