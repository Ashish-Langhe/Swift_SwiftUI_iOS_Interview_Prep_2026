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

## Deep Dive Expansion

### Why This Topic Matters In Real Apps

Closures Interview Guide is not only a syntax topic. In production Swift, it affects capture behavior, escaping lifetime, retain cycles, API ergonomics, and callback design. A junior developer should be able to recognize the syntax and explain the basic behavior. A mid-level developer should know when the feature improves clarity or safety. A senior developer should understand how it changes API design, testability, performance, or long-term maintenance.

In iOS work, this usually appears while button actions, animation completion, Combine callbacks, and async bridges. The important question is not "Can I write the syntax?" The important question is "Does this code make invalid states harder, make ownership clearer, and make future changes safer?"

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

Use this checklist when applying Closures Interview Guide in an app feature:

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

- Turn each answer into a story: situation, decision, tradeoff, result, and verification.
- For senior interviews, avoid only definitions. Explain why the design prevents bugs in a real app.
- For escaping closures, draw the ownership graph before choosing strong, weak, or unowned captures.
- A weak capture is not automatically correct; use it when the work should not keep the object alive.

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

1. Write a minimal example that shows Closures Interview Guide correctly.
2. Write a second example that demonstrates a common mistake.
3. Refactor the mistake into a safer version.
4. Explain the difference between the beginner solution and the production-ready solution.
5. Turn one answer into a real project story with tradeoffs and verification.

## Applied Day-Level Example

The following example is not the only way to use Closures Interview Guide, but it shows the kind of production shape you should connect this topic to:

```swift
final class BannerPresenter {
    var onDismiss: (() -> Void)?

    func show() {
        animateIn { [weak self] in
            self?.scheduleAutoDismiss()
        }
    }

    private func scheduleAutoDismiss() { }
    private func animateIn(completion: @escaping () -> Void) { completion() }
}
```

Closures are behavior values with lifetime. Once a closure escapes, think about who stores it, what it captures, and when it should be released.

### How To Study This Example

- Identify which part is syntax and which part is design.
- Ask what invalid state the example prevents.
- Ask what would need to change if the feature became async, generic, public, or shared across modules.
- Rewrite the example with one deliberate bug, then explain how a reviewer should catch it.
- Turn the example into a two-minute interview explanation with tradeoffs.

