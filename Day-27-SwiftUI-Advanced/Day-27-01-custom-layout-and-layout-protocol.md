# Day 27: Custom Layout And Layout Protocol

## Why Custom Layout Matters

Most SwiftUI screens can be built with `VStack`, `HStack`, `ZStack`, `Grid`, `List`, and lazy containers. Advanced SwiftUI starts when those are not expressive enough.

Use custom layout when:

- A design has non-standard placement rules.
- You need reusable layout behavior.
- `GeometryReader` solutions become brittle.
- You need layout to adapt based on child sizes.
- You want to avoid hard-coded frames.

## The Layout Mental Model

SwiftUI layout is a negotiation:

1. Parent proposes a size.
2. Child reports the size it wants.
3. Parent places the child.

The `Layout` protocol lets you participate in this process directly.

## Basic Custom Layout

```swift
struct FlowLayout: Layout {
    var spacing: CGFloat = 8

    func sizeThatFits(
        proposal: ProposedViewSize,
        subviews: Subviews,
        cache: inout ()
    ) -> CGSize {
        let maxWidth = proposal.width ?? 0
        var currentX: CGFloat = 0
        var currentY: CGFloat = 0
        var lineHeight: CGFloat = 0

        for subview in subviews {
            let size = subview.sizeThatFits(.unspecified)

            if currentX + size.width > maxWidth, currentX > 0 {
                currentX = 0
                currentY += lineHeight + spacing
                lineHeight = 0
            }

            currentX += size.width + spacing
            lineHeight = max(lineHeight, size.height)
        }

        return CGSize(width: maxWidth, height: currentY + lineHeight)
    }

    func placeSubviews(
        in bounds: CGRect,
        proposal: ProposedViewSize,
        subviews: Subviews,
        cache: inout ()
    ) {
        var currentX = bounds.minX
        var currentY = bounds.minY
        var lineHeight: CGFloat = 0

        for subview in subviews {
            let size = subview.sizeThatFits(.unspecified)

            if currentX + size.width > bounds.maxX, currentX > bounds.minX {
                currentX = bounds.minX
                currentY += lineHeight + spacing
                lineHeight = 0
            }

            subview.place(
                at: CGPoint(x: currentX, y: currentY),
                proposal: ProposedViewSize(size)
            )

            currentX += size.width + spacing
            lineHeight = max(lineHeight, size.height)
        }
    }
}
```

Use:

```swift
FlowLayout(spacing: 8) {
    ForEach(tags) { tag in
        Text(tag.title)
            .padding(.horizontal, 10)
            .padding(.vertical, 6)
            .background(.blue.opacity(0.12))
            .clipShape(Capsule())
    }
}
```

## Cache

The cache lets a layout avoid repeating expensive calculations.

```swift
struct CachedFlowLayout: Layout {
    struct Cache {
        var sizes: [CGSize] = []
    }

    func makeCache(subviews: Subviews) -> Cache {
        Cache(sizes: subviews.map { $0.sizeThatFits(.unspecified) })
    }

    func updateCache(_ cache: inout Cache, subviews: Subviews) {
        cache.sizes = subviews.map { $0.sizeThatFits(.unspecified) }
    }
}
```

Senior note: cache only when measurement cost is meaningful and correctness is clear.

## `AnyLayout`

Use `AnyLayout` to switch layouts without rebuilding view structure.

```swift
struct AdaptiveActions: View {
    @Environment(\.horizontalSizeClass) private var sizeClass

    var layout: AnyLayout {
        sizeClass == .compact
            ? AnyLayout(VStackLayout(spacing: 12))
            : AnyLayout(HStackLayout(spacing: 12))
    }

    var body: some View {
        layout {
            Button("Cancel", role: .cancel) {}
            Button("Save") {}
        }
    }
}
```

This preserves identity better than switching between separate `HStack` and `VStack` branches.

## Real iOS Use Case: Tag Cloud

Search filter screens commonly need wrapping chips.

```swift
struct FiltersSection: View {
    @Binding var selectedTags: Set<Tag.ID>
    let tags: [Tag]

    var body: some View {
        FlowLayout(spacing: 8) {
            ForEach(tags) { tag in
                FilterChip(
                    title: tag.title,
                    isSelected: selectedTags.contains(tag.id)
                ) {
                    toggle(tag.id)
                }
            }
        }
    }
}
```

This is cleaner than a fragile `GeometryReader`-based chip layout.

## Latest SwiftUI Notes

Recent SwiftUI updates continue expanding container behavior, including reordering and swipe actions for stacks, grids, and custom layouts. Advanced custom layouts must therefore preserve identity and participate cleanly in interaction.

## Common Mistakes

- Building custom layout before trying simple containers.
- Hard-coding device widths.
- Ignoring dynamic type.
- Recalculating expensive measurements unnecessarily.
- Using layout to hide poor state modeling.
- Breaking identity by changing layout structure unnecessarily.

## Senior iOS Engineer Perspective

Before using `Layout`, ask:

- Is the layout rule reusable?
- Can built-in containers solve it?
- Does it work with dynamic type and localization?
- Are child identities stable?
- Is measurement efficient?
- Does it work in previews with extreme content?

## Interview Notes

Junior:

Most SwiftUI layout uses stacks and grids.

Mid-level:

Custom layout is useful when built-in containers cannot express placement rules.

Senior:

I use the `Layout` protocol to encode reusable layout algorithms, avoid brittle geometry hacks, preserve identity, and support adaptive UI with measurable performance.

## Practice

1. Build a wrapping tag layout.
2. Use `AnyLayout` to switch between vertical and horizontal actions.
3. Add previews for long text and large dynamic type.
4. Explain layout proposal, measurement, and placement.
