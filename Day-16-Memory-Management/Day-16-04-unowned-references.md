# Day 16: Unowned References

## Core Idea

An unowned reference does not keep an object alive, but unlike weak, it is expected to always have a valid object.

```swift
final class CreditCard {
    unowned let customer: Customer

    init(customer: Customer) {
        self.customer = customer
    }
}
```

If you access an unowned reference after the object is deallocated, the app crashes.

## Beginner Mental Model

Unowned means:

```text
I do not own this object, but I promise it will still exist when I use it.
```

If that promise is wrong, your app crashes.

## Weak Vs Unowned

Use `weak` when the referenced object may become nil.

Use `unowned` when the referenced object should have the same or longer lifetime.

```swift
weak var delegate: SomeDelegate?
unowned let owner: Owner
```

Decision table:

| Situation | Use |
| --- | --- |
| Reference may become nil | `weak` |
| Reference must always exist and lifetime is guaranteed | `unowned` |
| You are not completely sure | `weak` |

## Unowned Optional

Swift supports unowned optional references.

```swift
unowned var nextCourse: Course?
```

This is advanced and still requires you to ensure the reference is valid or nil.

## Real iOS Use Cases

Unowned is less common than weak in app code.

Possible uses:

- Child object guaranteed not to outlive owner
- Closures where lifetime is strictly controlled
- Graph structures with clear ownership

In most UIKit/SwiftUI app code, `weak` is safer.

## Unowned In Closures

```swift
someClosure = { [unowned self] in
    self.finish()
}
```

This is safe only if the closure cannot outlive `self`.

Dangerous:

```swift
service.fetch { [unowned self] result in
    self.handle(result)
}
```

Network callbacks are asynchronous. `self` may disappear before completion, so `unowned` can crash.

## `unowned(unsafe)`

Swift also has `unowned(unsafe)`, which avoids runtime safety checks. It is almost never appropriate in normal iOS app code.

Modern Swift 6.2 strict memory safety diagnostics can flag unsafe constructs, including unsafe unowned usage, when strict memory safety is enabled.

## Junior-Level Interview Answer

An unowned reference does not keep an object alive and is not optional by default.

## Mid-Level Interview Answer

Use unowned only when the referenced object will live at least as long as the object holding the reference. If that assumption is wrong, the app crashes.

## Senior-Level Interview Answer

Unowned is an ownership assertion. It trades optional handling for a runtime trap if the lifetime model is wrong. I use it rarely in app code because lifecycles can change. For delegates and callbacks, weak is usually safer. I use unowned only when the relationship is structurally guaranteed and documented.

## Common Mistakes

- Using unowned because it avoids optional syntax.
- Using unowned in async callbacks.
- Using unowned where object lifetime is not guaranteed.
- Confusing unowned with weak.

## Interview Progression

Basic:

Unowned does not keep an object alive and is non-optional by default.

Intermediate:

Use unowned only when the referenced object has the same or longer lifetime.

Advanced:

Unowned is a lifetime assertion. It removes optional handling but introduces a crash if the assertion is wrong. In app code with UI, async callbacks, navigation, and cancellation, lifetimes often change, so weak is usually safer.

## Practice

1. Explain weak vs unowned.
2. Identify whether a delegate should be weak or unowned.
3. Explain why unowned can crash.
4. Review a closure and decide weak/unowned/strong.

## Deep Dive Expansion

### Why This Topic Matters In Real Apps

Unowned References is not only a syntax topic. In production Swift, it affects ownership graphs, ARC behavior, cycle prevention, deallocation proof, and leak debugging. A junior developer should be able to recognize the syntax and explain the basic behavior. A mid-level developer should know when the feature improves clarity or safety. A senior developer should understand how it changes API design, testability, performance, or long-term maintenance.

In iOS work, this usually appears while view model lifetimes, delegates, closures, timers, tasks, and caches. The important question is not "Can I write the syntax?" The important question is "Does this code make invalid states harder, make ownership clearer, and make future changes safer?"

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

Use this checklist when applying Unowned References in an app feature:

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

- Prove memory fixes with deinit logs, Memory Graph, and repeated lifecycle testing.
- Look for cycles through closures, delegates, tasks, timers, subscriptions, and caches.

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

1. Write a minimal example that shows Unowned References correctly.
2. Write a second example that demonstrates a common mistake.
3. Refactor the mistake into a safer version.
4. Explain the difference between the beginner solution and the production-ready solution.
5. Add one test case or reasoning check that proves the behavior.

## Applied Day-Level Example

The following example is not the only way to use Unowned References, but it shows the kind of production shape you should connect this topic to:

```swift
final class DetailsViewModel {
    var onUpdate: (() -> Void)?

    deinit {
        print("DetailsViewModel deallocated")
    }

    func bind() {
        onUpdate = { [weak self] in
            self?.refreshDerivedState()
        }
    }

    private func refreshDerivedState() { }
}
```

Memory management is easiest when you can draw the graph. If `self` owns the closure and the closure owns `self`, ARC cannot release either side.

### How To Study This Example

- Identify which part is syntax and which part is design.
- Ask what invalid state the example prevents.
- Ask what would need to change if the feature became async, generic, public, or shared across modules.
- Rewrite the example with one deliberate bug, then explain how a reviewer should catch it.
- Turn the example into a two-minute interview explanation with tradeoffs.

