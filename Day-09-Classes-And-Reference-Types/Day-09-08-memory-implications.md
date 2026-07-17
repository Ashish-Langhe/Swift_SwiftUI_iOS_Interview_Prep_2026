# Day 9: Memory Implications

## Core Idea

Classes are managed by ARC: Automatic Reference Counting.

```swift
final class User { }

var user: User? = User()
user = nil
```

When no strong references remain, the object is deallocated.

## Retain Cycles

```swift
final class Owner {
    var closure: (() -> Void)?

    func setup() {
        closure = { [weak self] in
            self?.doWork()
        }
    }

    func doWork() { }
}
```

Use `weak` to avoid cycles when appropriate.

## Modern Swift 6.x Notes

Reference types with mutable state are more delicate in concurrency. Actors are reference types too, but protect isolated mutable state.

## Interview Levels

Junior:

ARC manages class memory.

Senior:

Memory issues with classes often involve ownership, retain cycles, closure captures, and shared mutable state. I use weak references for delegates and capture lists for closures when needed.

## Quick Notes

- Classes use ARC
- Strong references keep objects alive
- Weak references avoid ownership
- Closures can capture strongly
- Deinit helps detect leaks

## Interview Depth

Junior answer: Classes are managed by ARC, which keeps objects alive while strong references exist.

Mid-level answer: Retain cycles happen when objects strongly reference each other. Use `weak` for delegates and capture lists for closures.

Senior answer: Memory management is ownership design. Decide who owns what. Use `weak` for back-references, avoid stored closures capturing owners strongly, and use Instruments or memory graph debugging when leaks appear.

iOS use case:

```swift
protocol CoordinatorDelegate: AnyObject { }

final class Coordinator {
    weak var delegate: CoordinatorDelegate?
}
```

Common mistakes:

- Strong delegates.
- Strong `self` in stored escaping closures.
- Using `unowned` without guaranteed lifetime.

Practice:

1. Create retain cycle example.
2. Fix with weak.
3. Explain ARC.

## Deep Dive Expansion

### Why This Topic Matters In Real Apps

Memory Implications is not only a syntax topic. In production Swift, it affects identity, shared mutable state, lifecycle, inheritance tradeoffs, and memory impact. A junior developer should be able to recognize the syntax and explain the basic behavior. A mid-level developer should know when the feature improves clarity or safety. A senior developer should understand how it changes API design, testability, performance, or long-term maintenance.

In iOS work, this usually appears while view controllers, coordinators, services, delegates, and long-lived managers. The important question is not "Can I write the syntax?" The important question is "Does this code make invalid states harder, make ownership clearer, and make future changes safer?"

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

Use this checklist when applying Memory Implications in an app feature:

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

- Prefer code that communicates intent at the call site, not only code that compiles.
- When a feature grows, revisit whether the type still owns the right responsibilities.

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

1. Write a minimal example that shows Memory Implications correctly.
2. Write a second example that demonstrates a common mistake.
3. Refactor the mistake into a safer version.
4. Explain the difference between the beginner solution and the production-ready solution.
5. Add one test case or reasoning check that proves the behavior.

## Applied Day-Level Example

The following example is not the only way to use Memory Implications, but it shows the kind of production shape you should connect this topic to:

```swift
final class SearchCoordinator {
    private let navigationController: UINavigationController
    private var childCoordinators: [AnyObject] = []

    init(navigationController: UINavigationController) {
        self.navigationController = navigationController
    }

    func start() {
        let viewController = SearchViewController()
        navigationController.pushViewController(viewController, animated: true)
    }
}
```

Classes are useful when identity and lifecycle matter. A coordinator is not just data; it owns navigation behavior and may keep child flows alive.

### How To Study This Example

- Identify which part is syntax and which part is design.
- Ask what invalid state the example prevents.
- Ask what would need to change if the feature became async, generic, public, or shared across modules.
- Rewrite the example with one deliberate bug, then explain how a reviewer should catch it.
- Turn the example into a two-minute interview explanation with tradeoffs.

