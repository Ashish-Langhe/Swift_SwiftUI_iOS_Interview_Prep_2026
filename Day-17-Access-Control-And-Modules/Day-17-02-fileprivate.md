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

## Deep Dive Expansion

### Why This Topic Matters In Real Apps

fileprivate is not only a syntax topic. In production Swift, it affects API surface, encapsulation, package boundaries, framework evolution, and source compatibility. A junior developer should be able to recognize the syntax and explain the basic behavior. A mid-level developer should know when the feature improves clarity or safety. A senior developer should understand how it changes API design, testability, performance, or long-term maintenance.

In iOS work, this usually appears while feature modules, design systems, SDK entry points, and internal helpers. The important question is not "Can I write the syntax?" The important question is "Does this code make invalid states harder, make ownership clearer, and make future changes safer?"

### Beginner To Senior Progression

Beginner level:

- Define the concept in plain language.
- Write the smallest working example.
- Recognize the compiler error when the feature is used incorrectly.
- Explain what happens at the call site.

Mid-level level:

- Choose this feature over a nearby alternative for a clear reason.
- Handle edge cases, nil/error/cancellation/performance concerns where relevant.
- Keep the code readable for the next developer.
- Write tests around the behavior, not just the implementation detail.

Senior level:

- Explain the design tradeoff and the failure mode it prevents.
- Understand how this feature behaves across module, actor, memory, or API boundaries.
- Design examples that scale from a small screen to a larger feature.
- Avoid exposing implementation details as permanent API.

### Production-Style Example Pattern

Use this checklist when applying fileprivate in an app feature:

1. Identify the owner of the data or behavior.
2. Decide whether the value should be mutable, immutable, optional, throwing, async, isolated, or private.
3. Keep the public surface small and intention-revealing.
4. Add one realistic failure path, not only the happy path.
5. Check whether the code is still understandable from the call site.

```swift
struct FeatureState: Equatable {
    var isLoading: Bool
    var message: String?
    var canRetry: Bool
}

func makeInitialState() -> FeatureState {
    FeatureState(isLoading: false, message: nil, canRetry: false)
}
```

This small pattern is intentionally simple: define the state, control mutation through a narrow function, and make the result easy to inspect in tests.

### Edge Cases To Think About

- What happens when the input is empty, nil, duplicated, delayed, or invalid?
- What happens when this code is called repeatedly from a scrolling list or fast-changing UI?
- Does this API expose too much mutable state?
- Does the implementation assume a specific ordering, lifetime, actor, or thread?
- Will this still be easy to test after the feature grows?

### Topic-Specific Senior Notes

- Start restrictive and widen access only when another boundary genuinely needs it.
- Treat public and open declarations as compatibility promises, not convenience keywords.

### Common Interview Follow-Ups

Be ready for these follow-ups:

- Why did you choose this approach instead of the simpler alternative?
- What bug could happen if this is implemented carelessly?
- How would this behave in a large codebase with multiple modules?
- How would you test this without relying on UI screenshots?
- What changes when this code becomes async, public, generic, or shared?

### Strong Interview Framing

A strong answer should sound like this:

```text
I understand the basic syntax, but I also think about ownership and boundaries. I choose the approach that keeps state valid, makes the call site clear, and avoids unnecessary coupling. In a production iOS app, I would also consider testing, cancellation or error behavior where relevant, and whether this should remain an implementation detail or become part of a public API.
```

### Extra Practice

1. Write a minimal example that shows fileprivate correctly.
2. Write a second example that demonstrates a common mistake.
3. Refactor the mistake into a safer version.
4. Explain the difference between the beginner solution and the production-ready solution.
5. Add one test case or reasoning check that proves the behavior.

## Applied Day-Level Example

The following example is not the only way to use fileprivate, but it shows the kind of production shape you should connect this topic to:

```swift
public struct LoginFeatureView: View {
    private let configuration: LoginConfiguration

    public init(configuration: LoginConfiguration) {
        self.configuration = configuration
    }

    public var body: some View {
        LoginContentView(configuration: configuration)
    }
}

private struct LoginContentView: View {
    let configuration: LoginConfiguration
    var body: some View { Text(configuration.title) }
}
```

Access control keeps framework API small. The app needs the feature entry point, not every helper view inside the feature.

### How To Study This Example

- Identify which part is syntax and which part is design.
- Ask what invalid state the example prevents.
- Ask what would need to change if the feature became async, generic, public, or shared across modules.
- Rewrite the example with one deliberate bug, then explain how a reviewer should catch it.
- Turn the example into a two-minute interview explanation with tradeoffs.

