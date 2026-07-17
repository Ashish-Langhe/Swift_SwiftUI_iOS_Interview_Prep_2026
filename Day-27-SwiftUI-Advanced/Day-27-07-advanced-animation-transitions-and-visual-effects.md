# Day 27: Advanced Animation, Transitions, And Visual Effects

## Advanced Animation Mindset

Advanced animation is not "make everything move." It is communication:

- Show continuity.
- Explain hierarchy changes.
- Confirm user actions.
- Make direct manipulation feel physical.
- Preserve context across navigation.

## Matched Geometry Effect

```swift
struct ProductGallery: View {
    @Namespace private var namespace
    @State private var selectedProduct: Product?

    var body: some View {
        ZStack {
            if let selectedProduct {
                ProductDetail(product: selectedProduct)
                    .matchedGeometryEffect(id: selectedProduct.id, in: namespace)
                    .onTapGesture {
                        withAnimation(.spring) {
                            self.selectedProduct = nil
                        }
                    }
            } else {
                ProductGrid { product in
                    ProductCard(product: product)
                        .matchedGeometryEffect(id: product.id, in: namespace)
                        .onTapGesture {
                            withAnimation(.spring) {
                                selectedProduct = product
                            }
                        }
                }
            }
        }
    }
}
```

Stable identity is essential.

## Custom Transitions

```swift
extension AnyTransition {
    static var slideAndFade: AnyTransition {
        .move(edge: .bottom)
        .combined(with: .opacity)
    }
}
```

Use:

```swift
if isShowingBanner {
    SuccessBanner()
        .transition(.slideAndFade)
}
```

## Phase Animation

Use phase animation for multi-step effects.

```swift
PhaseAnimator([0.8, 1.12, 1.0], trigger: isCompleted) { content, scale in
    content.scaleEffect(scale)
} animation: { _ in
    .spring(duration: 0.3)
}
```

This is useful for confirmation, badges, and small feedback.

## Keyframe Animation

Keyframes are useful when timing matters.

Conceptual example:

```swift
KeyframeAnimator(initialValue: AnimationValues()) { values in
    CheckmarkView()
        .scaleEffect(values.scale)
        .opacity(values.opacity)
} keyframes: { _ in
    KeyframeTrack(\.scale) {
        CubicKeyframe(1.2, duration: 0.2)
        CubicKeyframe(1.0, duration: 0.15)
    }

    KeyframeTrack(\.opacity) {
        LinearKeyframe(1, duration: 0.1)
    }
}
```

Use keyframes for choreographed effects, not basic state changes.

## Navigation Transitions

Modern SwiftUI includes richer navigation transition APIs. Use them when a route change benefits from visual continuity.

Senior caution:

- Do not make navigation feel slow.
- Keep transitions consistent.
- Test interruption and back navigation.
- Respect reduce motion.

## Visual Effects

Visual effects can react to geometry.

```swift
ProductCard(product: product)
    .visualEffect { content, proxy in
        content
            .scaleEffect(proxy.frame(in: .scrollView).minY < 100 ? 0.95 : 1.0)
    }
```

Use sparingly. Geometry-driven effects can become expensive in scrolling containers.

## Reduce Motion

```swift
@Environment(\.accessibilityReduceMotion) private var reduceMotion

withAnimation(reduceMotion ? nil : .spring) {
    isExpanded.toggle()
}
```

Advanced animation must respect user settings.

## Common Mistakes

- Animating layout churn in large lists.
- Unstable IDs with matched geometry.
- Using long transitions for common navigation.
- Ignoring reduce motion.
- Encoding business logic in animation callbacks.
- Relying on arbitrary delays.

## Senior iOS Engineer Perspective

Senior animation review asks:

- What user understanding does this improve?
- Is identity stable?
- Can it be interrupted?
- Does it respect accessibility?
- Does it profile well?
- Is the fallback acceptable?
- Is timing state-driven or delay-driven?

## Interview Notes

Junior:

Animations happen when SwiftUI state changes.

Mid-level:

Use matched geometry, transitions, phase/keyframe animations, and reduce-motion checks.

Senior:

I design animation as state communication, preserve identity, avoid performance-heavy effects, test interruption, and respect accessibility settings.

## Practice

1. Build a matched-geometry card expansion.
2. Add a custom banner transition.
3. Add reduce-motion fallback.
4. Explain why animation should not hide slow work.
