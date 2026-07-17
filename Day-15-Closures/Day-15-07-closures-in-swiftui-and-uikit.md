# Day 15: Closures In SwiftUI And UIKit

## SwiftUI

SwiftUI uses closures heavily.

```swift
Button("Save") {
    viewModel.save()
}
```

View builders are closure-based.

```swift
List(items) { item in
    Text(item.title)
}
```

## UIKit

UIKit uses closures in animations and modern APIs.

```swift
UIView.animate(withDuration: 0.3) {
    view.alpha = 0
}
```

Completion handlers are common in networking and coordination.

## Modern Swift 6.x Notes

Swift 6.2 approachable concurrency means many callback-based APIs can be wrapped or replaced with async/await. Closures passed across concurrency boundaries may need `@Sendable`.

## Interview Levels

Junior: Closures are used for button actions and callbacks.

Senior: In UI code, closures must respect lifetime, actor isolation, and memory ownership. SwiftUI closures often run on main actor context, while escaping UIKit callbacks require capture care.

## Quick Notes

- SwiftUI actions are closures.
- UIKit animations use closures.
- Callbacks can escape.
- Use weak captures when needed.

## Interview Depth

Junior answer: SwiftUI and UIKit use closures for actions, animations, and callbacks.

Mid-level answer: SwiftUI button actions and UIKit animation blocks are common closure examples. Network completions are escaping closures.

Senior answer: UI closures must respect lifetime and main-thread/main-actor rules. SwiftUI view closures are value-driven, while UIKit escaping callbacks often need capture lists to avoid leaks.

iOS use case:

```swift
Button("Refresh") {
    Task {
        await viewModel.refresh()
    }
}
```

Common mistakes: retain cycles in UIKit callbacks, doing long work directly in UI closure, ignoring actor isolation.

Practice: write Button closure, write animation closure, add weak self in callback.
