# Day 10: Associated Values

## Core Idea

Associated values attach data to enum cases.

```swift
enum LoadingState {
    case loading
    case loaded([String])
    case failed(String)
}
```

## Pattern Matching

```swift
switch state {
case .loaded(let items):
    print(items)
case .failed(let message):
    print(message)
case .loading:
    print("Loading")
}
```

## Interview Levels

Junior:

Associated values store extra data with enum cases.

Senior:

Associated values let each state carry exactly the data it needs, avoiding many optional properties.

## Quick Notes

- Case-specific data
- Excellent for state machines
- Use switch to extract values

## Interview Depth

Junior answer: Associated values store extra data inside enum cases.

Mid-level answer: Each enum case can carry the exact data it needs, such as loaded data or an error message.

Senior answer: Associated values prevent optional-state explosion. Instead of storing `data?`, `error?`, and `isLoading`, one enum can represent valid states precisely.

iOS use case:

```swift
enum ScreenState {
    case loading
    case loaded([Transaction])
    case failed(message: String)
}
```

Common mistakes:

- Confusing associated values with raw values.
- Adding associated values when a struct would be clearer.
- Using `default` and ignoring associated data.

Practice:

1. Model network state.
2. Extract associated value in switch.
3. Explain why it reduces optionals.

## Deep Dive Expansion

### Why This Topic Matters In Real Apps

Associated Values is not only a syntax topic. In production Swift, it affects finite state modeling, associated data, exhaustive switching, and invalid state prevention. A junior developer should be able to recognize the syntax and explain the basic behavior. A mid-level developer should know when the feature improves clarity or safety. A senior developer should understand how it changes API design, testability, performance, or long-term maintenance.

In iOS work, this usually appears while loading states, screen routes, API errors, and enum-driven UI. The important question is not "Can I write the syntax?" The important question is "Does this code make invalid states harder, make ownership clearer, and make future changes safer?"

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

Use this checklist when applying Associated Values in an app feature:

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

1. Write a minimal example that shows Associated Values correctly.
2. Write a second example that demonstrates a common mistake.
3. Refactor the mistake into a safer version.
4. Explain the difference between the beginner solution and the production-ready solution.
5. Add one test case or reasoning check that proves the behavior.

## Applied Day-Level Example

The following example is not the only way to use Associated Values, but it shows the kind of production shape you should connect this topic to:

```swift
enum Loadable<Value> {
    case idle
    case loading
    case loaded(Value)
    case failed(String)
}

func render(_ state: Loadable<[Product]>) -> String {
    switch state {
    case .idle: return "Idle"
    case .loading: return "Loading"
    case .loaded(let products): return "Products: \(products.count)"
    case .failed(let message): return message
    }
}
```

Enums prevent invalid combinations like `isLoading == true` while also having an error and stale data unless you intentionally model that state.

### How To Study This Example

- Identify which part is syntax and which part is design.
- Ask what invalid state the example prevents.
- Ask what would need to change if the feature became async, generic, public, or shared across modules.
- Rewrite the example with one deliberate bug, then explain how a reviewer should catch it.
- Turn the example into a two-minute interview explanation with tradeoffs.

