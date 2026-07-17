# Day 26: Custom Components

## What a Custom Component Is

A custom component is a reusable SwiftUI view that captures structure, styling, behavior, or domain meaning.

Examples:

- `PrimaryButton`
- `ProductRow`
- `ProfileAvatar`
- `EmptyStateView`
- `LoadingButton`
- `PriceLabel`
- `FilterChip`

Good components reduce duplication without hiding important behavior.

## Simple Component

```swift
struct PrimaryButton: View {
    let title: String
    let isLoading: Bool
    let action: () -> Void

    var body: some View {
        Button(action: action) {
            HStack {
                if isLoading {
                    ProgressView()
                }

                Text(title)
                    .fontWeight(.semibold)
            }
            .frame(maxWidth: .infinity)
        }
        .buttonStyle(.borderedProminent)
        .disabled(isLoading)
    }
}
```

Use:

```swift
PrimaryButton(title: "Save", isLoading: model.isSaving) {
    Task { await model.save() }
}
```

## Component API Design

Good component APIs are:

- Small
- Explicit
- Domain-aware
- Previewable
- Accessible
- Hard to misuse

Weak:

```swift
CustomButton(text: String, color: Color, radius: CGFloat, width: CGFloat)
```

Better:

```swift
PrimaryButton(title: "Continue", isLoading: isLoading, action: continue)
```

Expose intent, not every styling knob.

## Generic Content Components

Use `@ViewBuilder` when the component needs flexible content.

```swift
struct Card<Content: View>: View {
    let content: Content

    init(@ViewBuilder content: () -> Content) {
        self.content = content()
    }

    var body: some View {
        VStack(alignment: .leading, spacing: 12) {
            content
        }
        .padding()
        .background(.background)
        .clipShape(RoundedRectangle(cornerRadius: 8))
    }
}
```

Use:

```swift
Card {
    Text("Revenue")
        .font(.headline)
    Text("$12,450")
        .font(.title.bold())
}
```

## Domain Components

Domain components communicate business meaning.

```swift
struct OrderStatusBadge: View {
    let status: OrderStatus

    var body: some View {
        Text(status.title)
            .font(.caption.weight(.semibold))
            .padding(.horizontal, 8)
            .padding(.vertical, 4)
            .background(status.tint.opacity(0.15))
            .foregroundStyle(status.tint)
            .clipShape(Capsule())
            .accessibilityLabel("Order status: \(status.title)")
    }
}
```

This is better than repeating badge styling across screens.

## Binding Components

Use binding when the component edits parent-owned state.

```swift
struct FilterChip: View {
    let title: String
    @Binding var isSelected: Bool

    var body: some View {
        Button {
            isSelected.toggle()
        } label: {
            Text(title)
                .padding(.horizontal, 12)
                .padding(.vertical, 8)
        }
        .buttonStyle(.bordered)
        .tint(isSelected ? .blue : .secondary)
    }
}
```

## Style Protocols

For reusable styling, consider SwiftUI style protocols.

```swift
struct LoadingButtonStyle: ButtonStyle {
    func makeBody(configuration: Configuration) -> some View {
        configuration.label
            .frame(maxWidth: .infinity)
            .padding(.vertical, 12)
            .background(configuration.isPressed ? Color.blue.opacity(0.8) : Color.blue)
            .foregroundStyle(.white)
            .clipShape(RoundedRectangle(cornerRadius: 8))
    }
}
```

Use:

```swift
Button("Continue") {}
    .buttonStyle(LoadingButtonStyle())
```

Senior tip: use styles for visual behavior that should apply across many controls. Use custom views for domain-specific composition.

## Accessibility in Components

Components must carry accessibility semantics.

```swift
struct IconCounter: View {
    let systemImage: String
    let count: Int
    let label: String

    var body: some View {
        Label("\(count)", systemImage: systemImage)
            .accessibilityLabel("\(count) \(label)")
    }
}
```

Do not make every component a visual-only wrapper.

## Preview Matrix

```swift
#Preview("Primary") {
    VStack {
        PrimaryButton(title: "Save", isLoading: false) {}
        PrimaryButton(title: "Saving", isLoading: true) {}
    }
    .padding()
}
```

Preview:

- Normal
- Loading
- Disabled
- Long text
- Dark mode
- Large dynamic type

## Common Mistakes

- Over-abstracting before patterns repeat.
- Creating style knobs for everything.
- Hiding important actions inside components.
- Ignoring accessibility labels.
- Making components depend on global state.
- Using `AnyView` unnecessarily.
- Building components that cannot be previewed.

## Senior iOS Engineer Perspective

Senior component design balances reuse and clarity:

- Extract around meaning.
- Keep APIs intentional.
- Use bindings only for editing.
- Use callbacks for intents.
- Keep dependencies explicit.
- Make preview states cheap.
- Bake in accessibility.
- Avoid generic abstractions that erase domain meaning.

## Interview Notes

Junior:

Custom components are reusable SwiftUI views.

Mid-level:

Use properties, bindings, callbacks, and `@ViewBuilder` to design flexible components.

Senior:

I design component APIs around ownership and intent. I avoid style-parameter soup, include accessibility, support previews, and extract only when reuse or clarity is real.

## Practice

1. Build a reusable `PrimaryButton`.
2. Build a `Card` with `@ViewBuilder` content.
3. Add accessibility labels to a visual component.
4. Refactor repeated status badge UI into a domain component.
