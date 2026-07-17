# Day 15: Closures Interview Guide

## One-Minute Interview Answer

Closures are self-contained blocks of code that can be stored, passed, and executed later. They can capture values from surrounding scope. Non-escaping closures run before a function returns, while escaping closures can run later and require `@escaping`. Because closures capture strongly by default, stored escaping closures can create retain cycles, so capture lists like `[weak self]` are important. Closures are everywhere in SwiftUI and UIKit, from button actions to animations and callbacks.

## Modern Swift 6.x Notes

Swift 6.2 introduced `@concurrent` and approachable concurrency changes. Closures crossing concurrency boundaries may need `@Sendable`, and async/await often replaces callback closures for networking and long-running work.

## Common Traps

- Forgetting `@escaping`.
- Capturing `self` strongly in stored closures.
- Using `unowned` without guaranteed lifetime.
- Overusing autoclosure.
- Ignoring actor isolation in async closure work.

## Topic-By-Topic Deep Dive

### Closure Syntax

Closures are function values.

```swift
let isValidEmail: (String) -> Bool = { email in
    email.contains("@")
}
```

Short syntax is common:

```swift
let doubled = [1, 2, 3].map { $0 * 2 }
```

Senior answer:

Use shorthand when it improves signal. Use named parameters when the closure has business meaning or multiple values.

### Capturing Values

```swift
func makeGreeter(prefix: String) -> (String) -> String {
    { name in "\(prefix), \(name)" }
}
```

The closure captures `prefix`.

Senior caution:

Captures extend lifetimes. This matters when captured values are class instances, view models, or controllers.

### Escaping Vs Non-Escaping

```swift
func animate(completion: @escaping () -> Void) {
    DispatchQueue.main.asyncAfter(deadline: .now() + 0.3) {
        completion()
    }
}
```

Escaping closures can run after the function returns, so captured objects may live longer than expected.

### Autoclosures

```swift
func assertCondition(_ condition: @autoclosure () -> Bool) {
    if !condition() {
        print("Assertion failed")
    }
}
```

Senior caution:

Autoclosure hides closure creation at the call site. It is best for assertion/logging-style APIs, not general app logic.

### Retain Cycles

```swift
final class ProfileViewModel {
    var onRefresh: (() -> Void)?

    func configure() {
        onRefresh = { [weak self] in
            self?.refresh()
        }
    }

    func refresh() { }
}
```

Retain cycle pattern:

`self` owns closure, closure owns `self`.

### Capture Lists

```swift
service.load { [weak self] result in
    guard let self else { return }
    self.handle(result)
}
```

Use `weak` when `self` may disappear. Use `unowned` only when lifetime is guaranteed.

### Closures In SwiftUI And UIKit

SwiftUI:

```swift
Button("Save") {
    viewModel.save()
}
```

UIKit:

```swift
UIView.animate(withDuration: 0.3) {
    view.alpha = 0
}
```

Senior answer:

UI closures often interact with main-actor state. Escaping closures in services should be reviewed for captures and actor/thread expectations.

## Production Decision Table

| Need | Closure Concept |
| --- | --- |
| Inline behavior | Closure syntax |
| Use outside values | Capture |
| Run later | `@escaping` |
| Delay expression | `@autoclosure` |
| Prevent memory leak | Capture list |
| UI actions | SwiftUI/UIKit closures |

## Final Revision

- Closure = unnamed function block.
- Captures values.
- Escaping can run later.
- Capture lists manage memory.
- SwiftUI and UIKit use closures heavily.
- Async/await often replaces callbacks.
