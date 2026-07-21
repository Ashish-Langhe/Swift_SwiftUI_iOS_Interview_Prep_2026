# Day 36.11: Responder Chain, Gestures, And Events

UIKit input handling is built around events, hit testing, gesture recognizers, and the responder chain. Senior engineers need this knowledge for debugging taps not working, keyboard shortcuts, custom controls, overlapping views, scroll conflicts, and advanced interaction design.

## Touch Handling Mental Model

When the user touches the screen:

1. The system creates touch events.
2. UIKit performs hit testing to find the target view.
3. Gesture recognizers attached to views inspect touches.
4. If needed, touch methods and responder chain handling occur.

## Hit Testing

UIKit asks the view hierarchy which view should receive a touch.

Important methods:

- `point(inside:with:)`
- `hitTest(_:with:)`

Example: increase a small button's tap area.

```swift
final class ExpandedHitButton: UIButton {
    var hitTestInsets = UIEdgeInsets(top: -12, left: -12, bottom: -12, right: -12)

    override func point(inside point: CGPoint, with event: UIEvent?) -> Bool {
        let largerBounds = bounds.inset(by: hitTestInsets)
        return largerBounds.contains(point)
    }
}
```

Senior caution:

- `isHidden = true` views do not receive touches.
- `alpha < 0.01` views usually do not receive touches.
- `isUserInteractionEnabled = false` disables interaction.
- Parent bounds can clip hit testing unless overridden.

## Gesture Recognizers

Basic tap recognizer:

```swift
let tap = UITapGestureRecognizer(target: self, action: #selector(cardTapped))
cardView.addGestureRecognizer(tap)
cardView.isUserInteractionEnabled = true

@objc private func cardTapped() {
    coordinator.showDetails()
}
```

Common recognizers:

- `UITapGestureRecognizer`
- `UIPanGestureRecognizer`
- `UILongPressGestureRecognizer`
- `UIPinchGestureRecognizer`
- `UIRotationGestureRecognizer`
- `UIScreenEdgePanGestureRecognizer`

## Gesture State

For pan gestures:

```swift
@objc private func handlePan(_ recognizer: UIPanGestureRecognizer) {
    let translation = recognizer.translation(in: view)

    switch recognizer.state {
    case .began:
        startPosition = cardView.center
    case .changed:
        cardView.center = CGPoint(
            x: startPosition.x + translation.x,
            y: startPosition.y + translation.y
        )
    case .ended, .cancelled:
        finishOrResetCard()
    default:
        break
    }
}
```

Senior thinking: always handle `.cancelled`. System gestures, interruptions, or recognizer conflicts can cancel a gesture.

## Gesture Conflicts

Example: allow a custom horizontal pan and collection view scroll to recognize together.

```swift
extension GalleryViewController: UIGestureRecognizerDelegate {
    func gestureRecognizer(
        _ gestureRecognizer: UIGestureRecognizer,
        shouldRecognizeSimultaneouslyWith otherGestureRecognizer: UIGestureRecognizer
    ) -> Bool {
        true
    }
}
```

Or require one to fail:

```swift
singleTap.require(toFail: doubleTap)
```

Use cases:

- Single tap vs double tap.
- Interactive pop vs custom pan.
- Scroll view pan vs child gesture.
- Long press context menus.

## Responder Chain

The responder chain lets unhandled events move from one responder to the next.

Typical chain:

```text
UIView -> UIViewController -> parent view/controller -> UIWindow -> UIApplication
```

Useful for:

- Keyboard commands.
- Menu actions.
- Cut/copy/paste.
- Motion events.
- First responder management.

## First Responder

Text fields become first responder to receive keyboard input.

```swift
emailTextField.becomeFirstResponder()
emailTextField.resignFirstResponder()
```

Find useful keyboard flow:

```swift
func textFieldShouldReturn(_ textField: UITextField) -> Bool {
    switch textField {
    case emailTextField:
        passwordTextField.becomeFirstResponder()
    default:
        textField.resignFirstResponder()
        submit()
    }
    return true
}
```

## Keyboard Commands

For iPad keyboard support:

```swift
override var keyCommands: [UIKeyCommand]? {
    [
        UIKeyCommand(
            input: "f",
            modifierFlags: .command,
            action: #selector(focusSearch),
            discoverabilityTitle: "Search"
        )
    ]
}

@objc private func focusSearch() {
    searchField.becomeFirstResponder()
}
```

Senior note: keyboard support matters more on iPad, Mac Catalyst, and professional apps.

## Touch Methods

You can override low-level touch methods in custom views:

```swift
override func touchesBegan(_ touches: Set<UITouch>, with event: UIEvent?) {
    super.touchesBegan(touches, with: event)
    isHighlighted = true
}

override func touchesEnded(_ touches: Set<UITouch>, with event: UIEvent?) {
    super.touchesEnded(touches, with: event)
    isHighlighted = false
}
```

Prefer gesture recognizers for most interaction. Use touch overrides for custom controls or drawing surfaces.

## Scroll View Interactions

Scroll views already own pan gestures. Custom gestures inside scroll views require care.

Example:

```swift
func gestureRecognizerShouldBegin(_ gestureRecognizer: UIGestureRecognizer) -> Bool {
    guard let pan = gestureRecognizer as? UIPanGestureRecognizer else {
        return true
    }

    let velocity = pan.velocity(in: view)
    return abs(velocity.x) > abs(velocity.y)
}
```

This lets a horizontal gesture begin only when horizontal movement dominates.

## Common Bugs

- Button does not tap because parent has `isUserInteractionEnabled = false`.
- Gesture recognizer blocks table row selection.
- Overlay view intercepts touches.
- Tap area is visually too small.
- Keyboard never appears because view is not first responder.
- Interactive pop gesture conflicts with custom pan.

## Debugging Checklist

- Is the view hidden?
- Is alpha effectively zero?
- Is user interaction enabled on the view and ancestors?
- Is another view covering it?
- Does hit testing return the expected view?
- Is a gesture recognizer cancelling touches?
- Is the control inside a scroll view delaying touches?

## Junior-Level Interview Answer

UIKit uses hit testing to find which view gets a touch. Gesture recognizers detect taps, pans, and other gestures. The responder chain passes events upward if the first object does not handle them.

## Senior-Level Interview Answer

I debug input by separating hit testing, gesture recognition, and responder-chain behavior. I check view visibility, interaction flags, z-order, gesture delegate conflicts, scroll-view delays, and first responder state. For advanced interactions, I handle recognizer state transitions including cancellation, and I keep custom hit testing intentional.

## Points To Remember

- Hit testing chooses the touched view.
- Gestures interpret touch sequences.
- Responder chain handles unhandled actions.
- First responder receives keyboard input.
- Gesture cancellation is real and must be handled.
- Scroll views are gesture-heavy, so conflicts are common.

