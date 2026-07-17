# Day 11: Computed Properties

## Core Idea

Computed properties calculate values instead of storing them.

```swift
struct User {
    let firstName: String
    let lastName: String

    var fullName: String {
        "\(firstName) \(lastName)"
    }
}
```

## Getter And Setter

```swift
struct Temperature {
    var celsius: Double

    var fahrenheit: Double {
        get { celsius * 9 / 5 + 32 }
        set { celsius = (newValue - 32) * 5 / 9 }
    }
}
```

## Real iOS Use Cases

- Display labels
- Derived validation state
- UI enable/disable conditions

```swift
struct LoginForm {
    var email: String
    var password: String

    var canSubmit: Bool {
        email.contains("@") && password.count >= 8
    }
}
```

## Interview Levels

Junior: A computed property returns a calculated value.

Senior: Computed properties are good for cheap derived values. Expensive work, async work, or side effects should usually be methods so callers understand cost and behavior.

## Quick Notes

- No stored backing value by default.
- Can be read-only or get/set.
- Great for derived UI state.
- Avoid hidden expensive computation.

## Interview Depth

Junior answer: A computed property calculates a value when accessed.

Mid-level answer: Computed properties are great for derived values like `fullName`, `canSubmit`, or formatted state.

Senior answer: Use computed properties for cheap, synchronous, deterministic values. Expensive, throwing, or async work should usually be a method so cost is visible.

iOS use case:

```swift
var isSubmitEnabled: Bool {
    email.contains("@") && password.count >= 8
}
```

Common mistakes: doing network calls, sorting large arrays every access, hiding side effects.

Practice: create `fullName`, create `canSubmit`, decide property vs method.

## Deep Dive Expansion

### Why This Topic Matters In Real Apps

Computed Properties is not only a syntax topic. In production Swift, it affects state ownership, computed behavior, lazy initialization, observation, and wrapper semantics. A junior developer should be able to recognize the syntax and explain the basic behavior. A mid-level developer should know when the feature improves clarity or safety. A senior developer should understand how it changes API design, testability, performance, or long-term maintenance.

In iOS work, this usually appears while view models, cached values, validation state, and SwiftUI data flow. The important question is not "Can I write the syntax?" The important question is "Does this code make invalid states harder, make ownership clearer, and make future changes safer?"

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

Use this checklist when applying Computed Properties in an app feature:

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

1. Write a minimal example that shows Computed Properties correctly.
2. Write a second example that demonstrates a common mistake.
3. Refactor the mistake into a safer version.
4. Explain the difference between the beginner solution and the production-ready solution.
5. Add one test case or reasoning check that proves the behavior.

## Applied Day-Level Example

The following example is not the only way to use Computed Properties, but it shows the kind of production shape you should connect this topic to:

```swift
final class CartViewModel: ObservableObject {
    @Published private(set) var items: [CartItem] = []

    var total: Decimal {
        items.reduce(0) { $0 + $1.price }
    }

    func add(_ item: CartItem) {
        items.append(item)
    }
}
```

Properties are where state ownership becomes visible. Stored properties hold truth, computed properties derive truth, and restricted setters protect mutation rules.

### How To Study This Example

- Identify which part is syntax and which part is design.
- Ask what invalid state the example prevents.
- Ask what would need to change if the feature became async, generic, public, or shared across modules.
- Rewrite the example with one deliberate bug, then explain how a reviewer should catch it.
- Turn the example into a two-minute interview explanation with tradeoffs.

