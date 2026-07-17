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

## Deep Dive Expansion

### Why This Topic Matters In Real Apps

Weak References is not only a syntax topic. In production Swift, it affects ownership graphs, ARC behavior, cycle prevention, deallocation proof, and leak debugging. A junior developer should be able to recognize the syntax and explain the basic behavior. A mid-level developer should know when the feature improves clarity or safety. A senior developer should understand how it changes API design, testability, performance, or long-term maintenance.

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

Use this checklist when applying Weak References in an app feature:

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

1. Write a minimal example that shows Weak References correctly.
2. Write a second example that demonstrates a common mistake.
3. Refactor the mistake into a safer version.
4. Explain the difference between the beginner solution and the production-ready solution.
5. Add one test case or reasoning check that proves the behavior.

## Applied Day-Level Example

The following example is not the only way to use Weak References, but it shows the kind of production shape you should connect this topic to:

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

## Senior iOS Engineer Artifact

### Senior Mental Model

For a senior iOS engineer, the topic **Weak References** is evaluated through this lens: memory management is ownership proof; senior engineers verify lifecycle with evidence, not guesses. The goal is not to memorize the keyword or syntax in isolation. The goal is to understand how this topic changes correctness, ownership, testability, performance, and team-scale maintainability.

A senior engineer asks:

- What invariant does this protect?
- What future bug does this make harder to introduce?
- What does the call site communicate to another engineer?
- What is the runtime, memory, concurrency, or API-evolution cost?
- Can this design survive a larger feature, a second caller, or a module boundary?

### Artifact To Produce While Studying

Create this artifact for the topic:

```text
Artifact: a memory artifact with ownership graph, expected deinit points, known long-lived owners, and leak verification steps
Topic: Weak References
Owner: Which type/module owns the decision?
Boundary: Is this local, feature-wide, package-wide, public API, actor-isolated, or UI-only?
Invariant: What must always remain true?
Failure Mode: What breaks if this is misused?
Verification: How would I prove it with tests, logs, compiler checks, or tooling?
```

This artifact forces senior-level thinking. It turns a language feature into an engineering decision with evidence.

### Senior-Level Scenario

Imagine debugging leaked view models, coordinators, closures, timers, tasks, Combine subscriptions, and caches. A junior answer usually explains what the syntax does. A senior answer explains the design pressure:

```text
I would first identify the owner and boundary. Then I would choose the smallest surface that preserves the invariant, keeps the call site readable, and avoids leaking implementation details. I would also check the failure path and decide how to verify the behavior with tests or tooling.
```

### What Can Go Wrong

The senior-level risk for this area is fixing symptoms with weak everywhere instead of breaking the correct ownership edge.

That risk matters because iOS apps grow by accumulation. A small unclear decision can become a pattern copied into screens, services, tests, and public APIs.

### Review Checklist

Use this checklist in code review:

- What retains this object?
- When should deinit happen?
- How did we prove the leak is gone?
- Is the example still understandable six months later?
- Does the implementation make the safe path easier than the unsafe path?
- If this appears in an interview, can I explain both the syntax and the tradeoff?

### Senior Interview Framing

Use this structure when answering at senior level:

```text
At the syntax level, this feature does X. At the design level, I use it to protect Y. The tradeoff is Z. In a real iOS app, I would apply it in this scenario, verify it this way, and avoid this common misuse.
```

For **Weak References**, fill it like this:

```text
At the syntax level, this topic gives me a Swift mechanism for a specific behavior. At the design level, I use it to make ownership, state, or boundaries clearer. The tradeoff is that misuse can hide complexity or create coupling. In a real iOS app, I would apply it where the invariant matters, verify it with focused tests or tooling, and avoid using it just because the syntax is available.
```

## More Coding Examples

These examples are intentionally small, but they are shaped like real app code. Use them to connect **Weak References** to code you might write in a SwiftUI/UIKit feature.

### Example 1: Proving Deallocation

```swift
final class ProfileViewModel {
    deinit {
        print("ProfileViewModel deinit")
    }
}

var model: ProfileViewModel? = ProfileViewModel()
model = nil
```

A `deinit` log is a simple first proof when learning ARC behavior.

### Example 2: Breaking A Cycle

```swift
final class ChildCoordinator {
    weak var parent: ParentCoordinator?
}

final class ParentCoordinator {
    var child: ChildCoordinator?
}
```

One side of a parent-child back-reference should usually be weak.

### How To Extend These Examples

- Add one failure path.
- Add one test case.
- Add one version that would be wrong in production and explain why.
- Explain what changes if this code moves from one screen into a shared module.

## Topic-Focused Mini Example

### Avoid ownership from callback to owner

```swift
service.onUpdate = { [weak self] value in
    self?.render(value)
}
```

Weak captures are appropriate when the callback should not keep the screen alive.

### Why This Fits Weak References

This example is intentionally small so the core idea is easy to see. After understanding it, expand it with a failure path, a test case, and one realistic constraint from a production iOS feature.

