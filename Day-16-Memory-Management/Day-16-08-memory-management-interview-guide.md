# Day 16: Memory Management Interview Guide

## One-Minute Interview Answer

Swift uses ARC, Automatic Reference Counting, to manage class instance memory. Strong references keep objects alive, while weak and unowned references do not. Weak references become nil automatically and are used for delegates, back-references, and closure captures where the object may disappear. Unowned references are non-owning but assume the object is still alive, so they can crash if the lifetime assumption is wrong. The biggest iOS memory problems are retain cycles, especially delegates, parent-child relationships, timers, subscriptions, tasks, and closures that capture `self`. I debug memory issues with `deinit` logs, Xcode Memory Graph, and Instruments.

## Modern Swift 6.x Notes

ARC remains the core class memory management model. Swift 6.x adds important memory-safety context:

- Swift 6.2 introduced `Span` for safe access to contiguous memory.
- Swift 6.2 introduced opt-in strict memory safety diagnostics for unsafe constructs.
- Swift concurrency checking makes shared mutable reference state more explicit.
- Actors can protect mutable state, but actors are still reference types.

These features complement ARC; they do not replace it.

## Full Learning Path: Basic To Advanced

### Level 1: Basic Definitions

Know these cold:

- ARC manages class instance memory.
- Strong keeps an object alive.
- Weak does not keep an object alive and becomes nil.
- Unowned does not keep an object alive and can crash if used after deallocation.
- Retain cycle means objects strongly keep each other alive.

### Level 2: Practical iOS Ownership

Know where memory issues happen:

- Delegates
- Closures
- Timers
- Coordinators
- View models
- Combine subscriptions
- Async tasks
- Caches
- Large images and `Data`

### Level 3: Senior Ownership Design

Be able to draw and explain ownership:

```text
NavigationController
  -> ViewController
      -> ViewModel
          -> Service
```

Back-references should usually be weak:

```text
ChildCoordinator --weak--> ParentCoordinator
```

### Level 4: Advanced Swift 6.x Memory Safety

Understand the difference:

- ARC: object lifetime for class instances
- Exclusivity: prevents conflicting memory access
- Strict memory safety: flags unsafe constructs
- `Span`: safe contiguous memory access
- Concurrency checking: catches unsafe shared mutable state

These are related but not identical.

## ARC

ARC counts strong references to class instances.

```swift
var object: MyObject? = MyObject()
object = nil
```

When strong reference count reaches zero, ARC deallocates the object.

Interview point:

ARC is deterministic reference counting, not a tracing garbage collector.

Advanced point:

ARC cannot collect cycles because each object in the cycle still has a positive strong reference count. That is why ownership design matters.

## Strong References

Strong means ownership.

```swift
final class ViewModel {
    private let service: APIService
}
```

Use strong for dependencies and children that the object owns.

## Weak References

Weak means non-ownership and optional lifetime.

```swift
weak var delegate: PaymentDelegate?
```

Use weak for delegates and back-references.

## Unowned References

Unowned means non-ownership but expected valid lifetime.

```swift
unowned let owner: Owner
```

Senior warning:

Unowned is a runtime assertion. Prefer weak unless lifetime is guaranteed.

## Retain Cycles

Classic cycle:

```swift
final class A {
    var b: B?
}

final class B {
    var a: A?
}
```

Break one side with weak if it is a back-reference.

## Closures And Leaks

```swift
onUpdate = { [weak self] in
    self?.refresh()
}
```

Closures capture strongly by default. Stored escaping closures are the major leak source in iOS interviews.

## Debugging Memory Issues

Use this order:

1. Reproduce the screen lifecycle.
2. Add `deinit` logs.
3. Use Xcode Memory Graph.
4. Check strong reference paths.
5. Use Instruments Leaks/Allocations.
6. Fix ownership.
7. Retest deallocation.

## Interview Story Answer

If asked "How did you debug a memory leak?", answer like this:

```text
I reproduced the leak by opening and dismissing the screen several times.
I added deinit logs to the view controller and view model.
The view controller deallocated, but the view model did not.
Then I opened Xcode Memory Graph and found the view model was retained by a stored closure.
The closure captured self strongly.
I changed the closure to capture self weakly, reran the flow, verified deinit printed, and checked Allocations to confirm memory stopped growing.
```

That answer sounds practical and senior because it includes reproduction, evidence, root cause, fix, and verification.

## Junior Questions

What is ARC?

Automatic Reference Counting. It manages memory by counting references.

What is a retain cycle?

Two or more objects keep each other alive through strong references.

What is weak?

A reference that does not keep an object alive and becomes nil.

## Mid-Level Questions

Why are delegates weak?

Delegates are usually callbacks/back-references. The owner should not strongly own its delegate.

Why do closures leak?

Closures capture `self` strongly by default. If `self` stores the closure, a cycle can happen.

Weak vs unowned?

Weak is optional and safe when object may disappear. Unowned is non-optional and crashes if object is gone.

## Senior Questions

When should you not use `[weak self]`?

If the operation must keep `self` alive to complete correctly, a strong capture may be intentional. The capture choice should match ownership and lifecycle.

How do Swift 6 memory-safety features relate to ARC?

ARC manages class lifetimes. Strict memory safety diagnostics and `Span` target unsafe memory access patterns, especially low-level pointer-like code. Concurrency checking helps identify unsafe shared mutable state.

How do you debug a memory leak?

Define expected ownership, reproduce lifecycle, verify `deinit`, inspect retain graph, use Instruments, fix the ownership root cause, and retest.

## Common Interview Traps

- Saying ARC prevents all leaks.
- Saying weak and unowned are the same.
- Using unowned just to avoid optional handling.
- Forgetting closure capture cycles.
- Making delegates strong.
- Ignoring tasks, timers, and subscriptions.
- Debugging without a reproducible lifecycle.

## Memory Ownership Decision Table

| Relationship | Reference |
| --- | --- |
| Object owns dependency | Strong |
| Parent owns child | Strong |
| Child points back to parent | Weak |
| Delegate callback | Weak |
| Closure may outlive owner | Weak capture |
| Guaranteed same/longer lifetime | Unowned, rare |
| Shared mutable async state | Actor or main actor isolation |

## Advanced Decision Table

| Problem | Likely Cause | Tool |
| --- | --- | --- |
| View model not deallocating | Closure/delegate/subscription cycle | Memory Graph |
| Memory grows after repeated navigation | Retained screens/resources | Allocations |
| Crash after object disappears | `unowned` misuse | Crash logs / Memory Graph |
| Huge memory spikes | Images/Data buffers | Allocations |
| Race around shared object | Mutable reference state | Actor / `@MainActor` |
| Unsafe pointer warnings | Unsafe constructs | Strict memory safety |

## Final Senior-Level Answer

Memory management in Swift starts with ARC, but senior iOS memory work is ownership design. I use strong references for ownership, weak references for delegates and back-references, and unowned only when lifetime is guaranteed. I review closure captures carefully because stored escaping closures are a common source of leaks. For debugging, I do not guess: I reproduce the lifecycle, add `deinit` logs, inspect strong reference paths in Memory Graph, use Instruments for leaks and allocations, fix the ownership edge, and verify. With modern Swift, I also consider concurrency isolation, task cancellation, strict memory-safety diagnostics, and safer low-level APIs like `Span`.

## Final Revision

- ARC manages class instances.
- Strong keeps alive.
- Weak does not keep alive and becomes nil.
- Unowned does not keep alive and can crash.
- Retain cycles leak memory.
- Closures capture strongly by default.
- Debug with deinit, Memory Graph, and Instruments.
- Swift 6.x adds stricter memory-safety tools, not a replacement for ARC.
