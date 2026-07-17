# Day 9: Classes And Reference Types Interview Guide

## One-Minute Interview Answer

Classes are reference types in Swift. Multiple variables can point to the same object, so mutation through one reference is visible through others. Classes support identity, inheritance, overriding, deinitialization, and ARC-managed memory. I use classes when identity, shared mutable state, UIKit subclassing, or Objective-C interoperability is needed. For simple data models, structs are usually safer.

## Modern Swift 6.x Notes

Swift 6's concurrency model makes shared mutable reference types important to design carefully. Use `@MainActor` for UI-bound classes, actors for isolated mutable state, and immutable value snapshots where possible.

## Junior Questions

What is a class?

A reference type with properties and methods.

What is ARC?

Automatic Reference Counting manages class memory.

## Senior Questions

When choose class over struct?

When identity, shared mutation, inheritance, deinit, or framework requirements matter.

How avoid retain cycles?

Use weak delegates, capture lists, and clear ownership.

## Common Traps

- Using classes for every model
- Forgetting closure retain cycles
- Deep inheritance hierarchies
- Missing `final`
- Confusing `==` and `===`

## Topic-By-Topic Deep Dive

### Class Declaration

```swift
final class SessionManager {
    var currentUser: User?
}
```

Senior note:

Use `final` unless the class is intentionally designed for inheritance.

### Reference Semantics

```swift
let first = SessionManager()
let second = first
second.currentUser = user
```

Both references see the same object.

Senior answer:

Reference semantics are useful for identity and shared state, but they increase ownership and concurrency complexity.

### Identity Vs Equality

`===` checks same object instance. `==` checks value equality.

```swift
if controllerA === controllerB {
    print("Same screen object")
}
```

### Initializers And Deinitializers

`deinit` helps confirm lifetime:

```swift
deinit {
    print("ProfileViewModel deallocated")
}
```

If it never prints, investigate retain cycles.

### Inheritance

Inheritance is common in UIKit:

```swift
final class ProfileViewController: UIViewController { }
```

Senior note:

For app business logic, prefer composition unless a true subtype relationship exists.

### Method Overriding

```swift
override func viewDidLoad() {
    super.viewDidLoad()
}
```

Call `super` when framework contracts require it.

### Final

`final` documents that subclassing is not supported. It also gives the compiler optimization opportunities.

### Memory Implications

Classes use ARC. Watch strong reference cycles:

```swift
closure = { [weak self] in
    self?.refresh()
}
```

## Production Decision Table

| Need | Use |
| --- | --- |
| Identity | Class |
| Shared mutable service | Class or actor |
| UIKit screen | Class |
| Simple data | Struct |
| Prevent subclassing | `final` |
| Avoid retain cycle | `weak` / capture list |
| Isolated mutable state | Actor |

## Final Revision

- Classes are reference types
- `===` checks identity
- Inheritance needs `override`
- `final` prevents subclassing
- ARC manages memory
- Weak references avoid cycles

## Deep Dive Expansion

### Why This Topic Matters In Real Apps

Classes And Reference Types Interview Guide is not only a syntax topic. In production Swift, it affects identity, shared mutable state, lifecycle, inheritance tradeoffs, and memory impact. A junior developer should be able to recognize the syntax and explain the basic behavior. A mid-level developer should know when the feature improves clarity or safety. A senior developer should understand how it changes API design, testability, performance, or long-term maintenance.

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

Use this checklist when applying Classes And Reference Types Interview Guide in an app feature:

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

- Turn each answer into a story: situation, decision, tradeoff, result, and verification.
- For senior interviews, avoid only definitions. Explain why the design prevents bugs in a real app.

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

1. Write a minimal example that shows Classes And Reference Types Interview Guide correctly.
2. Write a second example that demonstrates a common mistake.
3. Refactor the mistake into a safer version.
4. Explain the difference between the beginner solution and the production-ready solution.
5. Turn one answer into a real project story with tradeoffs and verification.

## Applied Day-Level Example

The following example is not the only way to use Classes And Reference Types Interview Guide, but it shows the kind of production shape you should connect this topic to:

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

