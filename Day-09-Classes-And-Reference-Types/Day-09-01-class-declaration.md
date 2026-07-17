# Day 9: Class Declaration

## Core Idea

A class defines a reference type.

```swift
final class UserSession {
    var token: String?
}
```

Instances are references to shared objects.

## Real iOS Use Cases

- View controllers
- Services
- Coordinators
- Managers
- Shared session state

## Modern Swift 6.x Notes

Shared mutable class instances require more care with Swift concurrency. Use actors, main actor isolation, locks, or immutable snapshots when needed.

## Interview Levels

Junior:

A class is a type that can have properties and methods.

Senior:

Classes provide identity and reference semantics. I use them when shared identity matters, not just as a default model type.

## Quick Notes

- Class is reference type
- Supports inheritance
- Can have deinit
- Good for identity-based objects

## Interview Depth

Junior answer: A class is a type with properties and methods.

Mid-level answer: Classes are reference types, which means variables can point to the same object. Classes support inheritance and deinitializers.

Senior answer: Use classes when identity matters, when shared mutable state is intentional, or when a framework requires subclassing. Do not use classes for every model by default.

iOS use case:

```swift
@MainActor
final class ProfileViewModel {
    private(set) var state: ProfileState = .loading
}
```

Common mistakes:

- Not marking app classes `final`.
- Using classes for plain values.
- Sharing mutable class state across tasks.

Practice:

1. Declare a final service class.
2. Explain class vs struct.
3. Explain when identity matters.

## Deep Dive Expansion

### Why This Topic Matters In Real Apps

Class Declaration is not only a syntax topic. In production Swift, it affects identity, shared mutable state, lifecycle, inheritance tradeoffs, and memory impact. A junior developer should be able to recognize the syntax and explain the basic behavior. A mid-level developer should know when the feature improves clarity or safety. A senior developer should understand how it changes API design, testability, performance, or long-term maintenance.

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

Use this checklist when applying Class Declaration in an app feature:

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

1. Write a minimal example that shows Class Declaration correctly.
2. Write a second example that demonstrates a common mistake.
3. Refactor the mistake into a safer version.
4. Explain the difference between the beginner solution and the production-ready solution.
5. Add one test case or reasoning check that proves the behavior.

## Applied Day-Level Example

The following example is not the only way to use Class Declaration, but it shows the kind of production shape you should connect this topic to:

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

