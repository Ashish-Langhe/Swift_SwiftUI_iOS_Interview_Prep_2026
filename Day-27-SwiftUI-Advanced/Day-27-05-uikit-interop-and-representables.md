# Day 27: UIKit Interop And Representables

## Why UIKit Interop Still Matters

Even in modern SwiftUI apps, you may need UIKit for:

- Existing screens
- Custom controls
- Complex text editing
- Camera and media pickers
- Map or web integrations
- Third-party SDK views
- Incremental migration

Advanced SwiftUI engineers know both directions of interop.

## `UIViewRepresentable`

Wrap a UIKit view.

```swift
struct RatingView: UIViewRepresentable {
    @Binding var rating: Int

    func makeUIView(context: Context) -> UISegmentedControl {
        let control = UISegmentedControl(items: ["1", "2", "3", "4", "5"])
        control.addTarget(
            context.coordinator,
            action: #selector(Coordinator.changed(_:)),
            for: .valueChanged
        )
        return control
    }

    func updateUIView(_ uiView: UISegmentedControl, context: Context) {
        if uiView.selectedSegmentIndex != rating - 1 {
            uiView.selectedSegmentIndex = rating - 1
        }
    }

    func makeCoordinator() -> Coordinator {
        Coordinator(rating: $rating)
    }

    final class Coordinator: NSObject {
        var rating: Binding<Int>

        init(rating: Binding<Int>) {
            self.rating = rating
        }

        @objc func changed(_ sender: UISegmentedControl) {
            rating.wrappedValue = sender.selectedSegmentIndex + 1
        }
    }
}
```

The coordinator bridges UIKit events back to SwiftUI state.

## `UIViewControllerRepresentable`

Wrap a UIKit controller.

```swift
struct CameraPicker: UIViewControllerRepresentable {
    let onImagePicked: (UIImage) -> Void

    func makeUIViewController(context: Context) -> UIImagePickerController {
        let picker = UIImagePickerController()
        picker.delegate = context.coordinator
        picker.sourceType = .camera
        return picker
    }

    func updateUIViewController(_ uiViewController: UIImagePickerController, context: Context) {}

    func makeCoordinator() -> Coordinator {
        Coordinator(onImagePicked: onImagePicked)
    }

    final class Coordinator: NSObject, UIImagePickerControllerDelegate, UINavigationControllerDelegate {
        let onImagePicked: (UIImage) -> Void

        init(onImagePicked: @escaping (UIImage) -> Void) {
            self.onImagePicked = onImagePicked
        }
    }
}
```

## Diff Updates

`updateUIView` can run often. Make it idempotent.

Bad:

```swift
func updateUIView(_ uiView: UILabel, context: Context) {
    uiView.text = expensiveText()
    uiView.font = expensiveFont()
}
```

Better:

```swift
func updateUIView(_ uiView: UILabel, context: Context) {
    if uiView.text != text {
        uiView.text = text
    }

    if uiView.textColor != color {
        uiView.textColor = color
    }
}
```

## Hosting SwiftUI in UIKit

Use `UIHostingController`.

```swift
let controller = UIHostingController(rootView: ProfileView(userID: userID))
navigationController?.pushViewController(controller, animated: true)
```

This is useful for incremental migration.

## Communication Boundaries

Preferred communication:

- SwiftUI state into UIKit through representable properties.
- UIKit events out through coordinator callbacks or bindings.
- Shared services through dependency injection.

Avoid:

- UIKit directly mutating random SwiftUI global state.
- SwiftUI reaching deep into UIKit hierarchy.
- Representables that recreate UIKit views unnecessarily.

## Memory Management

Coordinators can create retain cycles.

```swift
final class Coordinator: NSObject {
    weak var owner: SomeUIKitObject?
}
```

If closures capture view models or controllers, use capture lists deliberately.

## Common Mistakes

- Doing too much work in `updateUIView`.
- Recreating UIKit views instead of updating them.
- Forgetting delegate/coordinator lifetime.
- Threading UIKit updates off the main actor.
- Leaking through coordinator closures.
- Making representables impossible to test.

## Senior iOS Engineer Perspective

Senior interop design asks:

- Is UIKit genuinely needed?
- What is the SwiftUI state boundary?
- Are updates idempotent?
- Are delegate events mapped cleanly?
- Is memory safe?
- Can this be migrated later?
- Does Instruments show long platform updates?

## Interview Notes

Junior:

Use representables to show UIKit views in SwiftUI.

Mid-level:

Use coordinators for delegates and bindings/callbacks to pass data back.

Senior:

I treat representables as boundary adapters. I make updates idempotent, avoid unnecessary UIKit churn, manage coordinator memory, and use hosting controllers for incremental migration.

## Practice

1. Wrap a UIKit segmented control.
2. Add a coordinator callback.
3. Make `updateUIView` diff-based.
4. Explain how to host SwiftUI inside UIKit.
