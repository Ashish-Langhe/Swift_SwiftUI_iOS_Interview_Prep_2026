# Day 16: Retain Cycles

## Core Idea

A retain cycle happens when objects keep each other alive through strong references.

```swift
final class Person {
    var apartment: Apartment?

    deinit {
        print("Person deallocated")
    }
}

final class Apartment {
    var tenant: Person?

    deinit {
        print("Apartment deallocated")
    }
}
```

If `Person` strongly owns `Apartment` and `Apartment` strongly owns `Person`, ARC cannot deallocate either.

## How To See The Cycle

```text
Person -> Apartment
Apartment -> Person
```

Both arrows are strong. ARC sees each object still has a strong owner, so neither reference count reaches zero.

## Fix With Weak

```swift
final class Apartment {
    weak var tenant: Person?
}
```

Now `Apartment` does not keep `Person` alive.

## Common Retain Cycle Patterns

### Parent And Child

Parent strongly owns child. Child should weakly reference parent.

```swift
final class ParentCoordinator {
    var child: ChildCoordinator?
}

final class ChildCoordinator {
    weak var parent: ParentCoordinator?
}
```

### Delegate

Owner should not strongly own delegate.

```swift
weak var delegate: PaymentDelegate?
```

### Closure

Object owns closure. Closure strongly captures object.

```swift
final class ViewModel {
    var onUpdate: (() -> Void)?

    func bind() {
        onUpdate = {
            self.refresh()
        }
    }
}
```

Fix:

```swift
onUpdate = { [weak self] in
    self?.refresh()
}
```

### Timer

Repeating timers can keep closures alive.

```swift
timer = Timer.scheduledTimer(withTimeInterval: 1, repeats: true) { [weak self] _ in
    self?.tick()
}
```

Invalidate timers when appropriate.

### Notification Observer

Block-based observers can retain captured objects.

```swift
observer = NotificationCenter.default.addObserver(
    forName: UIApplication.didBecomeActiveNotification,
    object: nil,
    queue: .main
) { [weak self] _ in
    self?.refresh()
}
```

If you store the observer token, remove it when it is no longer needed.

### Combine Subscription

```swift
cancellable = publisher
    .sink { [weak self] value in
        self?.handle(value)
    }
```

If `self` owns `cancellable`, and the sink closure owns `self`, you have a cycle.

### Task

```swift
task = Task { [weak self] in
    await self?.load()
}
```

Tasks can keep work alive. Cancel long-running tasks intentionally.

## Real iOS Use Cases

Retain cycles commonly appear in:

- Coordinators
- Delegates
- View models with callbacks
- Timers
- Notification observers
- Combine subscriptions
- Long-running tasks

## Debugging Flow

1. Add `deinit` logs to the suspected object.
2. Navigate to the screen.
3. Dismiss or pop the screen.
4. Check if `deinit` prints.
5. If not, open Memory Graph.
6. Search for the object.
7. Inspect the strong reference path.
8. Break the incorrect ownership edge.
9. Retest.

## Junior-Level Interview Answer

A retain cycle happens when two objects hold strong references to each other, so ARC cannot free them.

## Mid-Level Interview Answer

Retain cycles are fixed by breaking one strong reference using `weak` or `unowned`. Delegates are usually weak. Closures often use `[weak self]`.

## Senior-Level Interview Answer

Retain cycle debugging is ownership debugging. I identify who should own whom, then make back-references weak. I do not apply weak randomly; I preserve necessary lifetimes while breaking cycles. I confirm fixes with `deinit`, Xcode Memory Graph, and Instruments.

## Common Mistakes

- Strong delegates.
- Coordinator child strongly references parent.
- Timer strongly retains target.
- Closure properties capture `self`.
- Combine sink captures `self` without weak capture.

## Interview Progression

Basic:

A retain cycle is when objects keep each other alive.

Intermediate:

Break cycles using weak or unowned on the non-owning side.

Advanced:

Retain-cycle fixes require ownership analysis. The goal is not "add weak everywhere"; the goal is to make the ownership graph accurately represent lifecycle. Verify with `deinit`, Memory Graph, and Instruments.

## Practice

1. Create a two-object retain cycle.
2. Fix it with weak.
3. Create closure retain cycle.
4. Add deinit logs to verify release.

## Deep Dive Expansion

### Why This Topic Matters In Real Apps

Retain Cycles is not only a syntax topic. In production Swift, it affects ownership graphs, ARC behavior, cycle prevention, deallocation proof, and leak debugging. A junior developer should be able to recognize the syntax and explain the basic behavior. A mid-level developer should know when the feature improves clarity or safety. A senior developer should understand how it changes API design, testability, performance, or long-term maintenance.

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

Use this checklist when applying Retain Cycles in an app feature:

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

1. Write a minimal example that shows Retain Cycles correctly.
2. Write a second example that demonstrates a common mistake.
3. Refactor the mistake into a safer version.
4. Explain the difference between the beginner solution and the production-ready solution.
5. Add one test case or reasoning check that proves the behavior.

## Applied Day-Level Example

The following example is not the only way to use Retain Cycles, but it shows the kind of production shape you should connect this topic to:

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

For a senior iOS engineer, the topic **Retain Cycles** is evaluated through this lens: memory management is ownership proof; senior engineers verify lifecycle with evidence, not guesses. The goal is not to memorize the keyword or syntax in isolation. The goal is to understand how this topic changes correctness, ownership, testability, performance, and team-scale maintainability.

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
Topic: Retain Cycles
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

For **Retain Cycles**, fill it like this:

```text
At the syntax level, this topic gives me a Swift mechanism for a specific behavior. At the design level, I use it to make ownership, state, or boundaries clearer. The tradeoff is that misuse can hide complexity or create coupling. In a real iOS app, I would apply it where the invariant matters, verify it with focused tests or tooling, and avoid using it just because the syntax is available.
```

