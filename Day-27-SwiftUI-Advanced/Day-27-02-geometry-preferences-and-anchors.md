# Day 27: Geometry, Preferences, And Anchors

## Why This Topic Matters

Advanced SwiftUI often needs information to flow from child views up to parent views:

- Child size reporting
- Scroll offset tracking
- Sticky headers
- Tab underline positioning
- Matched overlays
- Custom container behavior

SwiftUI prefers one-way data flow from parent to child, so upward layout data uses preferences.

## `GeometryReader`

`GeometryReader` exposes container geometry.

```swift
struct HeroImage: View {
    var body: some View {
        GeometryReader { proxy in
            Image("hero")
                .resizable()
                .scaledToFill()
                .frame(width: proxy.size.width, height: proxy.size.height)
                .clipped()
        }
        .frame(height: 240)
    }
}
```

Important: `GeometryReader` takes all available space proposed by the parent. This can surprise beginners.

## Avoid GeometryReader Abuse

Bad pattern:

```swift
GeometryReader { proxy in
    VStack {
        // entire screen layout manually calculated
    }
}
```

Better:

- Use stacks, grids, and layout modifiers first.
- Use `GeometryReader` for specific measurement needs.
- Prefer custom `Layout` for reusable placement logic.

## PreferenceKey

A `PreferenceKey` lets children send values upward.

```swift
struct HeaderHeightKey: PreferenceKey {
    static var defaultValue: CGFloat = 0

    static func reduce(value: inout CGFloat, nextValue: () -> CGFloat) {
        value = max(value, nextValue())
    }
}
```

Child writes:

```swift
Text("Profile")
    .background {
        GeometryReader { proxy in
            Color.clear
                .preference(key: HeaderHeightKey.self, value: proxy.size.height)
        }
    }
```

Parent reads:

```swift
.onPreferenceChange(HeaderHeightKey.self) { height in
    headerHeight = height
}
```

## Real Use Case: Equal Width Buttons

```swift
struct WidthKey: PreferenceKey {
    static var defaultValue: CGFloat = 0

    static func reduce(value: inout CGFloat, nextValue: () -> CGFloat) {
        value = max(value, nextValue())
    }
}

struct EqualWidthActions: View {
    @State private var maxWidth: CGFloat = 0

    var body: some View {
        HStack {
            actionButton("Cancel")
            actionButton("Save")
        }
        .onPreferenceChange(WidthKey.self) { maxWidth = $0 }
    }

    private func actionButton(_ title: String) -> some View {
        Button(title) {}
            .frame(width: maxWidth == 0 ? nil : maxWidth)
            .background {
                GeometryReader { proxy in
                    Color.clear.preference(key: WidthKey.self, value: proxy.size.width)
                }
            }
    }
}
```

## Anchor Preferences

Anchor preferences are useful when the parent needs a child's bounds in a coordinate space.

Conceptual tab underline:

```swift
struct TabBoundsKey: PreferenceKey {
    static var defaultValue: [Tab: Anchor<CGRect>] = [:]

    static func reduce(
        value: inout [Tab: Anchor<CGRect>],
        nextValue: () -> [Tab: Anchor<CGRect>]
    ) {
        value.merge(nextValue(), uniquingKeysWith: { $1 })
    }
}
```

Child:

```swift
Text(tab.title)
    .anchorPreference(key: TabBoundsKey.self, value: .bounds) {
        [tab: $0]
    }
```

Parent overlay:

```swift
.overlayPreferenceValue(TabBoundsKey.self) { values in
    GeometryReader { proxy in
        if let anchor = values[selectedTab] {
            let rect = proxy[anchor]
            Capsule()
                .frame(width: rect.width, height: 3)
                .offset(x: rect.minX, y: rect.maxY)
        }
    }
}
```

## Scroll Offset Tracking

Scroll tracking is a common reason to use preferences, but it should be done carefully.

```swift
struct ScrollOffsetKey: PreferenceKey {
    static var defaultValue: CGFloat = 0

    static func reduce(value: inout CGFloat, nextValue: () -> CGFloat) {
        value = nextValue()
    }
}
```

Use geometry in a named coordinate space:

```swift
ScrollView {
    Color.clear
        .frame(height: 0)
        .background {
            GeometryReader { proxy in
                Color.clear.preference(
                    key: ScrollOffsetKey.self,
                    value: proxy.frame(in: .named("scroll")).minY
                )
            }
        }

    content
}
.coordinateSpace(name: "scroll")
.onPreferenceChange(ScrollOffsetKey.self) { offset in
    scrollOffset = offset
}
```

Senior warning: scroll offset changes frequently. Do not trigger expensive work on every value.

## Common Mistakes

- Using `GeometryReader` as a replacement for layout fundamentals.
- Forgetting `GeometryReader` expands.
- Updating state too frequently from preferences.
- Creating feedback loops between measurement and layout.
- Using screen size instead of container size.
- Not testing dynamic type and rotation.

## Senior iOS Engineer Perspective

Geometry and preferences are advanced escape hatches:

- Use them only when parent-child layout information truly needs to flow upward.
- Keep measurements local.
- Avoid high-frequency expensive state updates.
- Prefer `Layout` for reusable layout algorithms.
- Test with real device sizes and accessibility settings.

## Interview Notes

Junior:

`GeometryReader` gives size and position information.

Mid-level:

`PreferenceKey` lets child views report values to ancestors.

Senior:

I use geometry, preferences, and anchors for measured UI effects while avoiding feedback loops, performance issues, and brittle screen-size assumptions.

## Practice

1. Measure child height with a preference key.
2. Build an animated tab underline with anchor preferences.
3. Track scroll offset without doing heavy work.
4. Explain why `GeometryReader` can unexpectedly fill space.
