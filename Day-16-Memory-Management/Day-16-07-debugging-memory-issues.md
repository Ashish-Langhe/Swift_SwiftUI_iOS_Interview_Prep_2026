# Day 16: Debugging Memory Issues

## Types Of Memory Issues

Common iOS memory issues:

- Memory leaks
- Retain cycles
- Excessive memory growth
- Large image/data retention
- Caches that never evict
- Tasks/subscriptions that outlive screens
- Accessing deallocated objects through unsafe references

## Basic Vs Advanced Memory Problems

Basic ARC issue:

```text
Object does not deallocate because something still strongly references it.
```

Advanced memory issue:

```text
Object deallocates, but memory still grows because images, caches, tasks, or buffers are retained elsewhere.
```

Do not assume every memory issue is one retain cycle.

## First Debugging Artifact: Deinit Logs

Add a `deinit` log to confirm release.

```swift
final class ProfileViewModel {
    deinit {
        print("ProfileViewModel deallocated")
    }
}
```

If `deinit` does not print after dismissal, investigate ownership.

## Deinit Log Template Artifact

```swift
deinit {
    print("DEINIT:", String(describing: Self.self))
}
```

For view controllers:

```swift
deinit {
    print("DEINIT ProfileViewController")
}
```

For view models:

```swift
deinit {
    print("DEINIT ProfileViewModel")
}
```

## Xcode Memory Graph

Use Xcode's Memory Graph Debugger to inspect object references.

Checklist:

1. Run the app.
2. Navigate to the screen.
3. Dismiss the screen.
4. Open Memory Graph.
5. Search for the view controller/view model.
6. Inspect strong reference paths.

What to look for:

- Strong reference chain back to the object
- Closure retaining owner
- Timer or subscription retaining owner
- Unexpected singleton/cache references
- Objects that remain after screen dismissal

## Instruments

Use Instruments for deeper analysis:

- Leaks instrument
- Allocations instrument
- Time Profiler when memory growth causes performance issues

Suggested workflow:

```text
Allocations:
1. Record baseline.
2. Navigate repeatedly through flow.
3. Observe persistent growth.
4. Mark generation.
5. Inspect objects that survive.

Leaks:
1. Run leak detection.
2. Reproduce lifecycle.
3. Inspect leaked object graph.
```

## Debugging Retain Cycles

Look for:

- Strong delegate
- Closure property capturing `self`
- Parent-child strong cycle
- Timer retaining target
- Notification observer not removed
- Combine subscription retaining owner
- Task retaining owner

## Debugging Large Memory Growth

Look for:

- Images not downsampled
- Large arrays retained by cache
- `Data` stored longer than needed
- Repeated decoding without release
- Autorelease-heavy Objective-C APIs

Image-specific checklist:

```text
[ ] Are large images downsampled before display?
[ ] Are original full-size images retained unnecessarily?
[ ] Is image cache bounded?
[ ] Are images released when screen disappears?
[ ] Are repeated thumbnails regenerated instead of cached appropriately?
```

## Modern Swift 6.x Notes

Swift 6.2 introduced opt-in strict memory safety diagnostics for unsafe constructs. This helps identify use of unsafe APIs such as unsafe pointers or `unowned(unsafe)`. This is different from ARC leak debugging, but both belong to memory safety.

Swift 6.2 also introduced `Span`, which offers safe access to contiguous memory and helps avoid pointer-style memory bugs in systems-level code.

## Memory Debugging Checklist Artifact

Use this checklist during app review:

```text
Memory Review Checklist

[ ] Does the screen's view model deinit after dismissal?
[ ] Are delegates weak?
[ ] Do stored closures capture self weakly when needed?
[ ] Are timers invalidated?
[ ] Are tasks cancelled when no longer needed?
[ ] Are Combine subscriptions released?
[ ] Are image/data caches bounded?
[ ] Are large images downsampled?
[ ] Are notification observers cleaned up?
[ ] Are unsafe APIs avoided or isolated?
```

## Ownership Graph Artifact

Before fixing, draw the expected ownership graph.

```text
ProfileViewController
  -> ProfileViewModel
      -> ProfileService

ProfileService
  -> URLSession

ProfileViewModel
  x should not strongly retain ProfileViewController
```

Then compare this with Memory Graph.

## Interview Debugging Answer

Junior:

I would add `deinit` logs and check whether objects are released.

Mid-level:

I would use Xcode Memory Graph to find strong reference paths and Instruments Leaks/Allocations to observe memory growth.

Senior:

I would reproduce the lifecycle, define the expected ownership graph, verify deallocation with `deinit`, inspect retain paths with Memory Graph, and confirm with Instruments. Then I would fix ownership at the source, not just add weak everywhere. I would also review tasks, subscriptions, caches, and large resources.

## Common Mistakes

- Assuming every memory issue is a retain cycle.
- Adding weak references randomly.
- Forgetting caches and large resources.
- Ignoring tasks/subscriptions.
- Not reproducing the lifecycle before debugging.

## Interview Progression

Basic:

Use `deinit` logs to check whether objects are released.

Intermediate:

Use Memory Graph and Instruments to inspect retain paths, leaks, and allocations.

Advanced:

Debugging memory means proving ownership. I reproduce the lifecycle, define expected ownership, capture evidence, inspect retain paths, fix the incorrect owner, and retest. For memory growth, I also inspect caches, images, tasks, subscriptions, and large data buffers.

## Practice

1. Add deinit to a view model.
2. Create and fix a delegate cycle.
3. Use Memory Graph to inspect references.
4. Explain Leaks vs Allocations instruments.

## Deep Dive Expansion

### Why This Topic Matters In Real Apps

Debugging Memory Issues is not only a syntax topic. In production Swift, it affects ownership graphs, ARC behavior, cycle prevention, deallocation proof, and leak debugging. A junior developer should be able to recognize the syntax and explain the basic behavior. A mid-level developer should know when the feature improves clarity or safety. A senior developer should understand how it changes API design, testability, performance, or long-term maintenance.

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

Use this checklist when applying Debugging Memory Issues in an app feature:

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

1. Write a minimal example that shows Debugging Memory Issues correctly.
2. Write a second example that demonstrates a common mistake.
3. Refactor the mistake into a safer version.
4. Explain the difference between the beginner solution and the production-ready solution.
5. Add one test case or reasoning check that proves the behavior.

## Applied Day-Level Example

The following example is not the only way to use Debugging Memory Issues, but it shows the kind of production shape you should connect this topic to:

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

For a senior iOS engineer, the topic **Debugging Memory Issues** is evaluated through this lens: memory management is ownership proof; senior engineers verify lifecycle with evidence, not guesses. The goal is not to memorize the keyword or syntax in isolation. The goal is to understand how this topic changes correctness, ownership, testability, performance, and team-scale maintainability.

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
Topic: Debugging Memory Issues
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

For **Debugging Memory Issues**, fill it like this:

```text
At the syntax level, this topic gives me a Swift mechanism for a specific behavior. At the design level, I use it to make ownership, state, or boundaries clearer. The tradeoff is that misuse can hide complexity or create coupling. In a real iOS app, I would apply it where the invariant matters, verify it with focused tests or tooling, and avoid using it just because the syntax is available.
```

