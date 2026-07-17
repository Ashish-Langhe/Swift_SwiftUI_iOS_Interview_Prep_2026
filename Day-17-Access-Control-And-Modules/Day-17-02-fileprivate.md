# Day 17: `fileprivate`

## What `fileprivate` Means

`fileprivate` allows a declaration to be used anywhere in the same source file.

```swift
fileprivate final class LoginValidator {
    func isValid(email: String, password: String) -> Bool {
        email.contains("@") && password.count >= 8
    }
}

struct LoginViewModel {
    private let validator = LoginValidator()
}
```

Any code in this file can use `LoginValidator`, but code in another file cannot.

## Beginner Mental Model

Use `fileprivate` when you want to say:

```text
This helper belongs to this file, but multiple declarations in this file need it.
```

It is wider than `private` but narrower than `internal`.

## `private` vs `fileprivate`

```swift
final class A {
    private var privateValue = 1
    fileprivate var fileValue = 2
}

final class B {
    func read(_ a: A) {
        // a.privateValue is not accessible here.
        print(a.fileValue) // Accessible if B is in the same file.
    }
}
```

Use `fileprivate` carefully. If too many unrelated types share file-private details, the file can become tightly coupled.

## Real iOS Use Cases

`fileprivate` is useful for:

- Helper types used only inside one file
- File-level constants
- Test-only helpers inside a test file
- UIKit delegate adapter types colocated with a view controller
- SwiftUI preview data inside a view file

```swift
struct PaymentView: View {
    let model: PaymentViewModel

    var body: some View {
        Text(model.displayTitle)
    }
}

fileprivate extension PaymentViewModel {
    var displayTitle: String {
        "\(merchantName) - \(amount.formatted())"
    }
}
```

This keeps formatting near the view without exposing it to the whole module.

## File-Scoped Helpers

```swift
final class OrderViewController: UIViewController {
    private let tableAdapter = OrderTableAdapter()
}

fileprivate final class OrderTableAdapter: NSObject, UITableViewDataSource {
    func tableView(_ tableView: UITableView, numberOfRowsInSection section: Int) -> Int {
        0
    }

    func tableView(
        _ tableView: UITableView,
        cellForRowAt indexPath: IndexPath
    ) -> UITableViewCell {
        UITableViewCell()
    }
}
```

The adapter is not part of the module API. It is an implementation detail of this file.

## When `fileprivate` Is Better Than `private`

Use it when two sibling declarations need to cooperate.

```swift
struct TimelineView: View {
    fileprivate let layout = TimelineLayout()

    var body: some View {
        Text("Timeline")
    }
}

fileprivate struct TimelineLayout {
    let spacing: CGFloat = 12
}
```

This is reasonable if `TimelineLayout` is not reusable outside this file.

## When `fileprivate` Is A Smell

Be cautious if a file has many types all reaching into each other's details.

```swift
final class CheckoutViewModel {
    fileprivate var mutableState = CheckoutState()
}

final class CheckoutLogger {
    func log(_ viewModel: CheckoutViewModel) {
        print(viewModel.mutableState)
    }
}
```

This may hide coupling instead of designing a clean interface.

Better:

```swift
final class CheckoutViewModel {
    private var mutableState = CheckoutState()

    var analyticsSnapshot: CheckoutAnalyticsSnapshot {
        CheckoutAnalyticsSnapshot(step: mutableState.step)
    }
}
```

## Module Impact

`fileprivate` never crosses a file boundary.

It is invisible to:

- Other files in the same app target
- Other targets in the same package
- Other modules
- Framework consumers

That makes it useful for implementation details that should stay very local.

## Advanced Design Notes

Senior engineers use `fileprivate` sparingly.

Good usage:

- Helps organize one cohesive source file.
- Hides helper types from the rest of the module.
- Supports previews or local adapters.

Weak usage:

- Avoids creating proper APIs.
- Allows unrelated types to mutate each other.
- Turns one file into a hidden mini-module with unclear boundaries.

## Modern Swift 6.x Notes

`fileprivate` remains stable in Swift 6.x.

The main modern consideration is concurrency:

- `fileprivate` does not make data safe across tasks.
- Access level and actor isolation are separate concepts.
- File visibility should not be used as a substitute for ownership design.

```swift
fileprivate var sharedCounter = 0

// This is hidden from other files, but still unsafe if accessed from many tasks.
```

Better:

```swift
fileprivate actor CounterStore {
    private var value = 0

    func increment() {
        value += 1
    }
}
```

## Junior Interview Answer

`fileprivate` means a declaration can be accessed anywhere in the same Swift file, but not from other files.

## Mid-Level Interview Answer

I use `fileprivate` for helper types or extensions that are only useful inside one file. I prefer `private` when only one type needs access.

## Senior Interview Answer

`fileprivate` is a tactical tool for file-local collaboration. It can reduce module API surface, but overuse can hide coupling. If several important types need file-private access to each other's state, I usually re-check the design and expose a smaller intentional interface instead.

## Points To Remember

- `fileprivate` is broader than `private`.
- It is still narrower than `internal`.
- It works across all declarations in the same source file.
- It is useful for local helper types.
- Overuse can create hidden coupling.

## Practice

1. Create a file-private formatter used by one view.
2. Convert an overused `fileprivate` property into `private` plus a computed snapshot.
3. Explain why `fileprivate` does not help with concurrency safety.
4. Identify whether a helper should be `private`, `fileprivate`, or `internal`.
