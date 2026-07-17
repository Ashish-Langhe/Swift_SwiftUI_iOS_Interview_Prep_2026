# Day 26: Animations

## What Animation Means in SwiftUI

SwiftUI animations are state transitions.

You change state, and SwiftUI animates the visual difference.

```swift
struct FavoriteButton: View {
    @State private var isFavorite = false

    var body: some View {
        Button {
            withAnimation(.spring) {
                isFavorite.toggle()
            }
        } label: {
            Image(systemName: isFavorite ? "heart.fill" : "heart")
                .foregroundStyle(isFavorite ? .pink : .secondary)
                .scaleEffect(isFavorite ? 1.2 : 1.0)
        }
    }
}
```

The animation is attached to the state change.

## Implicit Animation

```swift
RoundedRectangle(cornerRadius: isExpanded ? 24 : 8)
    .frame(height: isExpanded ? 180 : 80)
    .animation(.easeInOut, value: isExpanded)
```

This animates changes when `isExpanded` changes.

Use `value:` intentionally. It tells SwiftUI which state change should trigger the animation.

## Explicit Animation

```swift
withAnimation(.snappy) {
    selectedTab = .profile
}
```

Explicit animation is useful when a user action should animate a group of changes.

## Transitions

Transitions animate insertion and removal.

```swift
if isShowingBanner {
    Text("Saved")
        .padding()
        .background(.green)
        .transition(.move(edge: .top).combined(with: .opacity))
}
```

Trigger:

```swift
withAnimation {
    isShowingBanner = true
}
```

## Matched Geometry Effect

Use matched geometry for shared-element transitions.

```swift
@Namespace private var namespace
@State private var selectedProduct: Product?

var body: some View {
    if let selectedProduct {
        ProductDetail(product: selectedProduct)
            .matchedGeometryEffect(id: selectedProduct.id, in: namespace)
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
```

This is powerful but sensitive to identity and layout.

## Phase Animations and Keyframes

For multi-step animation, use phase or keyframe style APIs where available.

Conceptual example:

```swift
PhaseAnimator([0.8, 1.1, 1.0], trigger: isSaved) { content, scale in
    content
        .scaleEffect(scale)
} animation: { _ in
    .spring(duration: 0.35)
}
```

Use these when a state change has multiple visual phases.

## Animation Completion

Modern SwiftUI supports completion callbacks for animated changes in supported APIs.

Use cases:

- Remove temporary overlay after animation.
- Start next step after transition.
- Coordinate navigation after collapse.

Senior tip: avoid building fragile timing logic with arbitrary `DispatchQueue.main.asyncAfter` if completion APIs or state machines can express the flow.

## Gesture-Driven Animation

```swift
struct SwipeCard: View {
    @State private var offset: CGSize = .zero

    var body: some View {
        ProductCard()
            .offset(offset)
            .gesture(
                DragGesture()
                    .onChanged { value in
                        offset = value.translation
                    }
                    .onEnded { value in
                        withAnimation(.spring) {
                            offset = value.translation.width > 120
                                ? CGSize(width: 500, height: 0)
                                : .zero
                        }
                    }
            )
    }
}
```

Gesture animations should feel responsive and respect velocity where appropriate.

## Reduce Motion

Respect accessibility settings.

```swift
struct ExpandingCard: View {
    @Environment(\.accessibilityReduceMotion) private var reduceMotion
    @State private var isExpanded = false

    var body: some View {
        CardContent()
            .frame(height: isExpanded ? 240 : 120)
            .animation(reduceMotion ? nil : .spring, value: isExpanded)
    }
}
```

Animation should help comprehension, not block usage.

## Common Mistakes

- Animating everything.
- Using animation to hide slow UI.
- Forgetting stable identity.
- Using arbitrary delays instead of state.
- Ignoring reduce motion.
- Animating layout-heavy changes in large lists without profiling.
- Adding `.animation` too high in the tree.

## Senior iOS Engineer Perspective

Senior animation design asks:

- What state changed?
- Does animation clarify the change?
- Is identity stable?
- Is the animation accessible?
- Does it hurt scrolling or interaction?
- Can it be interrupted?
- Does it behave with real data?

## Latest SwiftUI Notes

Recent SwiftUI releases added richer animation tools over time: animation completion, phase/keyframe animation, symbol effects, navigation transitions, and newer spring behavior. The key senior move is still the same: tie animation to meaningful state transitions.

## Interview Notes

Junior:

Use `withAnimation` or `.animation(_:value:)` to animate state changes.

Mid-level:

Use transitions for insertion/removal, matched geometry for shared elements, and respect reduce motion.

Senior:

I design animations as state transitions, keep identity stable, avoid over-animation, profile list-heavy animations, and support accessibility and interruption.

## Practice

1. Animate expanding and collapsing a card.
2. Add a transition for a success banner.
3. Add reduce-motion behavior.
4. Explain why unstable IDs break animations.
