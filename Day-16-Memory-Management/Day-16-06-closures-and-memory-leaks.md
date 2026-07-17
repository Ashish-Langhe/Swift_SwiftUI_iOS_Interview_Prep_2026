# Day 16: Closures And Memory Leaks

## Why Closures Leak

Closures are reference types. When a closure captures a class instance, it captures strongly by default.

```swift
final class ProfileViewModel {
    var onReload: (() -> Void)?

    func setup() {
        onReload = {
            self.load()
        }
    }

    func load() { }
}
```

The view model owns `onReload`, and `onReload` owns `self`. This is a cycle.

## Closure Capture Rules

By default:

- Capturing a class instance is strong.
- Capturing a struct captures value semantics.
- Escaping closures can outlive the function call.
- Stored closures often outlive the immediate scope.

This is why escaping and stored closures need special attention.

## Fix With Capture List

```swift
onReload = { [weak self] in
    self?.load()
}
```

## Weak Self With Guard

```swift
service.fetch { [weak self] result in
    guard let self else { return }
    self.handle(result)
}
```

This pattern is common when the closure has multiple lines.

## Capture Values Instead Of Self

Sometimes you do not need `self`.

```swift
let userId = self.userId
service.fetchUser(id: userId) { result in
    print(result)
}
```

Capturing only the needed value reduces lifetime coupling.

## Capture Lists For Values

```swift
var count = 0

let printCount = { [count] in
    print(count)
}

count = 10
printCount() // prints 0
```

The capture list captured the value at closure creation time.

Senior point:

Capture lists are not only for `[weak self]`; they can intentionally freeze values.

## Timers

Timers can create cycles.

```swift
timer = Timer.scheduledTimer(withTimeInterval: 1, repeats: true) { [weak self] _ in
    self?.tick()
}
```

Invalidate timers when no longer needed.

## Tasks

Swift concurrency tasks can also affect object lifetime.

```swift
task = Task { [weak self] in
    await self?.load()
}
```

Cancel long-running tasks when appropriate.

## `@Sendable` And Modern Swift

Closures that cross concurrency boundaries may need to be `@Sendable`.

```swift
let operation: @Sendable () -> Void = {
    print("Safe to send across concurrency domains")
}
```

`@Sendable` is more about data-race safety than retain cycles, but both matter when closures run asynchronously.

## Advanced Closure Ownership Questions

Ask these during review:

1. Is the closure stored?
2. Is the closure escaping?
3. Does the closure capture `self`?
4. Does `self` own the closure?
5. Should the closure keep `self` alive?
6. Should the work cancel when `self` disappears?
7. Is the closure crossing actor or task boundaries?

## SwiftUI And UIKit Examples

UIKit callback:

```swift
UIView.animate(withDuration: 0.3) { [weak self] in
    self?.view.alpha = 0
}
```

SwiftUI action:

```swift
Button("Refresh") {
    Task {
        await viewModel.refresh()
    }
}
```

SwiftUI often uses value-type views, but reference-type view models can still leak through closures or tasks.

## Junior-Level Interview Answer

Closures can cause memory leaks because they capture objects strongly by default.

## Mid-Level Interview Answer

If an object stores a closure and that closure captures the object, a retain cycle can happen. Use `[weak self]` or `[unowned self]` depending on lifetime.

## Senior-Level Interview Answer

Closure memory management is about capture ownership. I ask whether the closure should keep `self` alive. For UI callbacks and optional lifetimes, weak is common. For operations that must complete and intentionally own their context, strong capture may be correct. I also consider tasks, timers, Combine subscriptions, and cancellation.

## Common Mistakes

- Using `[weak self]` without handling nil.
- Using `[unowned self]` in async work.
- Capturing all of `self` when only one value is needed.
- Forgetting to cancel tasks or invalidate timers.

## Interview Progression

Basic:

Closures can capture objects and keep them alive.

Intermediate:

Stored escaping closures that capture `self` can create retain cycles. Use capture lists to control ownership.

Advanced:

Closure lifetime must match business semantics. Weak capture is correct when work should stop if the owner disappears. Strong capture may be intentional when the operation owns its context. In concurrent Swift, also consider `@Sendable`, actor isolation, and cancellation.

## Practice

1. Create a closure property that leaks.
2. Fix it with `[weak self]`.
3. Capture a value instead of self.
4. Explain closure lifetime in a network callback.

## Deep Dive Expansion

### Why This Topic Matters In Real Apps

Closures And Memory Leaks is not only a syntax topic. In production Swift, it affects ownership graphs, ARC behavior, cycle prevention, deallocation proof, and leak debugging. A junior developer should be able to recognize the syntax and explain the basic behavior. A mid-level developer should know when the feature improves clarity or safety. A senior developer should understand how it changes API design, testability, performance, or long-term maintenance.

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

Use this checklist when applying Closures And Memory Leaks in an app feature:

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

- For escaping closures, draw the ownership graph before choosing strong, weak, or unowned captures.
- A weak capture is not automatically correct; use it when the work should not keep the object alive.
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

1. Write a minimal example that shows Closures And Memory Leaks correctly.
2. Write a second example that demonstrates a common mistake.
3. Refactor the mistake into a safer version.
4. Explain the difference between the beginner solution and the production-ready solution.
5. Add one test case or reasoning check that proves the behavior.

## Applied Day-Level Example

The following example is not the only way to use Closures And Memory Leaks, but it shows the kind of production shape you should connect this topic to:

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

For a senior iOS engineer, the topic **Closures And Memory Leaks** is evaluated through this lens: memory management is ownership proof; senior engineers verify lifecycle with evidence, not guesses. The goal is not to memorize the keyword or syntax in isolation. The goal is to understand how this topic changes correctness, ownership, testability, performance, and team-scale maintainability.

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
Topic: Closures And Memory Leaks
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

For **Closures And Memory Leaks**, fill it like this:

```text
At the syntax level, this topic gives me a Swift mechanism for a specific behavior. At the design level, I use it to make ownership, state, or boundaries clearer. The tradeoff is that misuse can hide complexity or create coupling. In a real iOS app, I would apply it where the invariant matters, verify it with focused tests or tooling, and avoid using it just because the syntax is available.
```

