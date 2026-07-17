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
