# Day 16: Weak References

## Core Idea

A weak reference does not keep an object alive.

Weak references are always optional because ARC automatically sets them to `nil` when the referenced object is deallocated.

## Beginner Mental Model

Weak means:

```text
I know about this object, but I do not own it.
```

If the object goes away, the weak reference becomes `nil`.

```swift
final class DelegateOwner {
    weak var delegate: LoginDelegate?
}
```

## Why Weak Exists

Weak references prevent ownership cycles.

```swift
protocol LoginDelegate: AnyObject {
    func didLogin()
}

final class LoginViewController {
    weak var delegate: LoginDelegate?
}
```

The view controller should not own its delegate.

## Delegate Ownership Pattern

Classic UIKit-style ownership:

```text
Parent/ViewController strongly owns Child
Child weakly references Delegate/Parent
```

Example:

```swift
protocol ChildViewControllerDelegate: AnyObject {
    func childDidFinish(_ child: ChildViewController)
}

final class ChildViewController: UIViewController {
    weak var delegate: ChildViewControllerDelegate?
}
```

The child should notify the parent, not own the parent.

## Weak Requires Class Types

Protocols used with `weak` must be class-constrained.

```swift
protocol PaymentDelegate: AnyObject {
    func paymentDidComplete()
}
```

`AnyObject` means only class types can conform.

## Real iOS Use Cases

Use `weak` for:

- Delegates
- Back-references from child to parent
- Closure captures where `self` may disappear
- UIKit relationships that should not own

```swift
final class ChildCoordinator {
    weak var parent: ParentCoordinator?
}
```

## Weak In Closures

```swift
service.loadProfile { [weak self] result in
    guard let self else { return }
    self.handle(result)
}
```

Use `[weak self]` when the closure may outlive `self` and should not keep `self` alive.

## Weak-Strong Dance

```swift
service.load { [weak self] result in
    guard let self else { return }
    self.handle(result)
}
```

This means:

1. Capture `self` weakly so the closure does not keep it alive.
2. Temporarily create a strong `self` while the closure executes.
3. Exit if `self` is already gone.

## When Weak Is Wrong

Weak is not always correct.

```swift
uploader.upload(file) { [weak self] result in
    self?.markUploadFinished(result)
}
```

If `self` disappearing should cancel or ignore the result, weak is fine. If the upload must complete and update required state, weak may silently skip important work. In that case, make ownership explicit through an operation object, task owner, or strong capture with lifecycle control.

## Junior-Level Interview Answer

A weak reference does not keep an object alive. It becomes nil when the object is deallocated.

## Mid-Level Interview Answer

Weak references are used to avoid retain cycles, especially with delegates and closure captures. Weak references must be optional and can only be used with class types.

## Senior-Level Interview Answer

Weak means non-ownership. It is not a default fix for every closure; it should match the ownership model. If a closure must finish work that requires `self`, weak capture may silently skip needed work. If `self` should stay alive until completion, a strong capture may be intentional. The right answer depends on lifecycle and ownership.

## Common Mistakes

- Forgetting `AnyObject` on delegate protocols.
- Using weak for dependencies that should be strongly owned.
- Blindly using `[weak self]` everywhere.
- Not handling the nil case after weak capture.

## Interview Progression

Basic:

Weak references do not keep objects alive and become nil automatically.

Intermediate:

Use weak for delegates, back-references, and closures that should not extend object lifetime.

Advanced:

Weak is an ownership decision, not a reflex. It prevents cycles but can also cause work to be skipped if the object disappears. Senior engineers choose weak, strong, or unowned based on desired lifetime semantics.

## Practice

1. Create a delegate protocol with `AnyObject`.
2. Explain why delegate should be weak.
3. Use `[weak self]` in an escaping closure.
4. Explain when weak capture can be wrong.
