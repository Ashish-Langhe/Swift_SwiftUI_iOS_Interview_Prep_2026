# Day 26: Gestures And Interactions

## Why Gestures Matter

SwiftUI gestures let you build direct manipulation:

- Taps
- Long presses
- Dragging
- Swiping
- Pinching
- Custom controls
- Card interactions
- Reorder-like behavior

Gesture code is easy to make flashy and hard to make correct. Intermediate skill means tying gestures to state, accessibility, and cancellation behavior.

## Tap Gesture

```swift
ProductCard(product: product)
    .onTapGesture {
        open(product.id)
    }
```

For most tappable controls, prefer `Button` because it carries accessibility and interaction semantics.

```swift
Button {
    open(product.id)
} label: {
    ProductCard(product: product)
}
.buttonStyle(.plain)
```

## Long Press

```swift
ProductCard(product: product)
    .onLongPressGesture {
        isShowingActions = true
    }
```

Long press should usually reveal secondary actions, not primary required behavior.

## Drag Gesture

```swift
struct DraggableCard: View {
    @State private var offset: CGSize = .zero

    var body: some View {
        RoundedRectangle(cornerRadius: 12)
            .fill(.blue)
            .frame(width: 180, height: 120)
            .offset(offset)
            .gesture(
                DragGesture()
                    .onChanged { value in
                        offset = value.translation
                    }
                    .onEnded { value in
                        withAnimation(.spring) {
                            offset = value.translation.width > 120
                                ? CGSize(width: 500, height: value.translation.height)
                                : .zero
                        }
                    }
            )
    }
}
```

Gesture state should usually be separate from committed model state.

## `@GestureState`

Use `@GestureState` for temporary gesture values.

```swift
struct PressableCard: View {
    @GestureState private var isPressed = false

    var body: some View {
        ProductCard()
            .scaleEffect(isPressed ? 0.97 : 1.0)
            .gesture(
                LongPressGesture(minimumDuration: 0.1)
                    .updating($isPressed) { value, state, _ in
                        state = value
                    }
            )
            .animation(.snappy, value: isPressed)
    }
}
```

`@GestureState` resets automatically when the gesture ends.

## Combining Gestures

```swift
let drag = DragGesture()
    .onChanged { value in
        dragOffset = value.translation
    }

let longPress = LongPressGesture()
    .onEnded { _ in
        isEditing = true
    }

content
    .gesture(longPress.sequenced(before: drag))
```

Common combinations:

- Simultaneous
- Sequenced
- Exclusive

Senior tip: combined gestures can conflict with scrolling. Test inside real containers.

## Content Shape

```swift
HStack {
    Text(product.name)
    Spacer()
}
.contentShape(Rectangle())
.onTapGesture {
    open(product.id)
}
```

`contentShape` defines the tappable area. Without it, only visible content may receive taps.

## Gesture vs Button

Use `Button` when the interaction is an action.

Use gestures when direct manipulation matters.

Bad:

```swift
Text("Delete")
    .onTapGesture { delete() }
```

Better:

```swift
Button("Delete", role: .destructive) {
    delete()
}
```

## Accessibility

Custom gestures need accessible alternatives.

```swift
DraggableCard()
    .accessibilityAction(named: "Dismiss") {
        dismiss()
    }
```

If the only way to perform an action is a hidden drag gesture, many users cannot use it.

## Common Mistakes

- Using gestures where buttons are appropriate.
- Gesture interactions that conflict with scroll views.
- No accessible alternative.
- Committing model state during every drag update.
- Magic thresholds without testing.
- Forgetting animation and cancellation behavior.

## Senior iOS Engineer Perspective

Senior gesture design asks:

- Is this an action or direct manipulation?
- What state is temporary vs committed?
- Does it conflict with scrolling?
- What happens if the gesture cancels?
- Is there a VoiceOver or keyboard alternative?
- Are thresholds based on layout and user expectation?

## Interview Notes

Junior:

SwiftUI has gestures like tap, long press, and drag.

Mid-level:

Use `@GestureState` for temporary gesture state and `contentShape` to define touch areas.

Senior:

I treat gestures as stateful interactions with accessibility and cancellation requirements. I prefer `Button` for actions and gestures for direct manipulation.

## Practice

1. Convert a tappable text gesture into a `Button`.
2. Add a drag-to-dismiss card with cancellation.
3. Add an accessibility action for a custom gesture.
4. Debug a drag gesture inside a scroll view.
