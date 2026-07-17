# Day 16: ARC

## What ARC Means

ARC stands for Automatic Reference Counting. It is Swift's memory management system for class instances.

ARC automatically:

- Counts strong references to class instances
- Keeps an object alive while at least one strong reference exists
- Deallocates the object when no strong references remain

```swift
final class UserSession {
    let id: String

    init(id: String) {
        self.id = id
        print("Session created")
    }

    deinit {
        print("Session destroyed")
    }
}

var session: UserSession? = UserSession(id: "s1")
session = nil
```

When `session` becomes `nil`, the object has no strong references, so ARC can deallocate it.

## Beginner Mental Model

Think of every class instance as an object with an invisible ownership counter.

```text
UserSession object
strong reference count: 1
```

Every strong reference increases the count. Removing a strong reference decreases it. When the count reaches zero, ARC calls `deinit` and releases the object.

```swift
var first: UserSession? = UserSession(id: "s1") // count +1
var second = first                              // count +1

first = nil                                    // count -1
second = nil                                   // count -1, deinit can run
```

ARC is predictable because object lifetime follows references.

## ARC Is Not Garbage Collection

Garbage collection usually runs later and scans memory to find unused objects. ARC releases objects as soon as the strong reference count reaches zero.

Interview answer:

Swift ARC is deterministic reference counting, not tracing garbage collection. This gives predictable object lifetimes, but ARC cannot automatically solve strong reference cycles.

## ARC Applies To Classes

ARC manages reference types, mainly class instances.

Structs and enums are value types and are not reference-counted in the same way.

```swift
struct UserValue {
    let name: String
}

final class UserObject {
    let name: String

    init(name: String) {
        self.name = name
    }
}
```

`UserValue` is copied by value. `UserObject` is shared by reference and managed by ARC.

## Value Types Can Still Contain References

ARC does not manage value types like class instances, but value types can store references.

```swift
struct ImageRow {
    let title: String
    let loader: ImageLoader
}

final class ImageLoader { }
```

`ImageRow` is a value type, but `loader` is a class reference. Copying `ImageRow` copies the reference to the same loader object.

Senior interview point:

Value type does not always mean there is no reference behavior anywhere inside. You must inspect stored properties too.

## Real iOS Use Cases

ARC manages:

- `UIViewController`
- View models implemented as classes
- Services
- Coordinators
- Delegates
- Closures that capture class instances

```swift
final class ProfileViewModel {
    deinit {
        print("ProfileViewModel released")
    }
}
```

Adding `deinit` logs is a simple first step when checking if an object is leaking.

## ARC And Object Lifetimes In iOS

Typical healthy ownership chain:

```text
NavigationController
  strongly owns ViewController
    strongly owns ViewModel
      strongly owns Service
```

Problem ownership chain:

```text
ViewController
  strongly owns ViewModel
    strongly owns Closure
      strongly captures ViewController
```

This cycle prevents ARC from reaching a zero strong reference count.

## Advanced ARC Intuition

ARC inserts retain and release operations automatically. You normally do not write those operations yourself.

Important consequences:

- Passing class instances around can affect reference counts.
- Storing a class instance strongly keeps it alive.
- Capturing a class instance in an escaping closure keeps it alive.
- `weak` and `unowned` do not increase the strong reference count.

ARC is automatic, but ownership design is still your job.

## Modern Swift 6.x Notes

ARC is still the core memory management model for class instances. Modern Swift adds more safety around memory access and unsafe constructs:

- Swift 6.2 introduced opt-in strict memory safety diagnostics for unsafe APIs.
- Swift 6.2 introduced `Span` for safe access to contiguous memory.
- Swift 6 concurrency checking makes shared mutable reference state more visible and more constrained.

These do not replace ARC. They complement it.

## Junior-Level Interview Answer

ARC automatically manages memory for class instances by counting references. When no strong references remain, the object is deallocated.

## Mid-Level Interview Answer

ARC works with strong, weak, and unowned references. Strong references keep objects alive. Weak and unowned references do not. ARC prevents most manual memory management issues, but retain cycles can still leak memory.

## Senior-Level Interview Answer

ARC is deterministic reference-counting for object lifetimes. It is not a garbage collector. Objects are released when their strong reference count reaches zero. The main production risks are ownership mistakes: strong reference cycles, closure captures, global/shared references, and long-lived caches. In modern Swift, I also consider concurrency isolation because shared mutable class instances can create both lifetime and data-race problems.

## Common Mistakes

- Thinking ARC prevents all memory leaks.
- Forgetting closures capture strongly by default.
- Making delegates strong.
- Keeping objects alive through singleton caches.
- Assuming `deinit` will run while a strong reference still exists.

## Interview Progression

Basic:

ARC counts references and frees class instances when no strong references remain.

Intermediate:

Strong references keep objects alive. Weak and unowned references do not. Retain cycles can leak memory because reference counts never reach zero.

Advanced:

ARC gives deterministic object lifetime, but ownership graphs must be designed carefully. I verify lifetimes with `deinit`, inspect retain paths with Memory Graph, and pay special attention to closures, delegates, timers, tasks, subscriptions, and caches. Swift 6.x strict memory safety helps unsafe low-level memory access, but ARC ownership issues still need architectural discipline.

## Practice

1. Create a class with `deinit` and observe when it prints.
2. Create two strong references to the same object.
3. Explain why ARC is not garbage collection.
4. Explain why value types are not managed like class instances.

## Deep Dive Expansion

### Why This Topic Matters In Real Apps

ARC is not only a syntax topic. In production Swift, it affects ownership graphs, ARC behavior, cycle prevention, deallocation proof, and leak debugging. A junior developer should be able to recognize the syntax and explain the basic behavior. A mid-level developer should know when the feature improves clarity or safety. A senior developer should understand how it changes API design, testability, performance, or long-term maintenance.

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

Use this checklist when applying ARC in an app feature:

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

1. Write a minimal example that shows ARC correctly.
2. Write a second example that demonstrates a common mistake.
3. Refactor the mistake into a safer version.
4. Explain the difference between the beginner solution and the production-ready solution.
5. Add one test case or reasoning check that proves the behavior.

## Applied Day-Level Example

The following example is not the only way to use ARC, but it shows the kind of production shape you should connect this topic to:

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

For a senior iOS engineer, the topic **ARC** is evaluated through this lens: memory management is ownership proof; senior engineers verify lifecycle with evidence, not guesses. The goal is not to memorize the keyword or syntax in isolation. The goal is to understand how this topic changes correctness, ownership, testability, performance, and team-scale maintainability.

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
Topic: ARC
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

For **ARC**, fill it like this:

```text
At the syntax level, this topic gives me a Swift mechanism for a specific behavior. At the design level, I use it to make ownership, state, or boundaries clearer. The tradeoff is that misuse can hide complexity or create coupling. In a real iOS app, I would apply it where the invariant matters, verify it with focused tests or tooling, and avoid using it just because the syntax is available.
```

