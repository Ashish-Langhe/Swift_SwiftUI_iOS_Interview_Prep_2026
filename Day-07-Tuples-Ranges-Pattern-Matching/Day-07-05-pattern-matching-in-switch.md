# Day 7: Pattern Matching In Switch

## Core Idea

Swift `switch` matches patterns, not just simple values.

```swift
let point = (0, 10)

switch point {
case (0, 0):
    print("Origin")
case (0, _):
    print("Y-axis")
case (_, 0):
    print("X-axis")
default:
    print("Other")
}
```

## Patterns

- Values
- Ranges
- Tuples
- Enums
- Optionals
- Type casts

## Modern Swift 6.x Notes

Swift 6.3 added `@c` support for exposing Swift enums to C in interop scenarios. For normal iOS interviews, focus on Swift enum pattern matching and exhaustiveness.

## Interview Levels

Junior:

Switch checks a value against cases.

Senior:

Swift switch is a pattern matching tool with exhaustiveness checking, making it excellent for enum state and domain modeling.

## Quick Notes

- Switch must be exhaustive
- Pattern order matters
- Supports tuples and ranges
- Great with enums

## Interview Depth

Junior answer: `switch` compares a value with cases.

Mid-level answer: Swift `switch` supports more than simple equality. It can match ranges, tuples, optionals, enums, and conditions with `where`.

Senior answer: Pattern matching is a modeling tool. With enums and associated values, it lets the compiler enforce exhaustive handling of app states. Avoid `default` for app-owned enums when explicit cases would catch future changes.

iOS use case:

```swift
switch state {
case .loading:
    showLoading()
case .loaded(let user):
    show(user)
case .failed(let error):
    show(error)
}
```

Common mistakes:

- Using `default` too early.
- Writing overlapping cases in confusing order.
- Using strings instead of enums.

Practice:

1. Switch over tuple point.
2. Switch over enum state.
3. Explain exhaustiveness.

## Deep Dive Expansion

### Why This Topic Matters In Real Apps

Pattern Matching In Switch is not only a syntax topic. In production Swift, it affects temporary grouping, expressive matching, boundary handling, and readable branching. A junior developer should be able to recognize the syntax and explain the basic behavior. A mid-level developer should know when the feature improves clarity or safety. A senior developer should understand how it changes API design, testability, performance, or long-term maintenance.

In iOS work, this usually appears while pagination ranges, validation results, and switch-driven state handling. The important question is not "Can I write the syntax?" The important question is "Does this code make invalid states harder, make ownership clearer, and make future changes safer?"

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

Use this checklist when applying Pattern Matching In Switch in an app feature:

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

1. Write a minimal example that shows Pattern Matching In Switch correctly.
2. Write a second example that demonstrates a common mistake.
3. Refactor the mistake into a safer version.
4. Explain the difference between the beginner solution and the production-ready solution.
5. Add one test case or reasoning check that proves the behavior.

## Applied Day-Level Example

The following example is not the only way to use Pattern Matching In Switch, but it shows the kind of production shape you should connect this topic to:

```swift
func pageRange(page: Int, pageSize: Int, total: Int) -> Range<Int> {
    let start = max(0, page * pageSize)
    let end = min(total, start + pageSize)
    return start..<end
}

let response = (status: 200, itemCount: 24)

switch response {
case (200..<300, let count) where count > 0:
    print("Render list")
case (200..<300, 0):
    print("Show empty state")
default:
    print("Show error")
}
```

Tuples and ranges are strongest when they make temporary relationships obvious without creating unnecessary permanent types.

### How To Study This Example

- Identify which part is syntax and which part is design.
- Ask what invalid state the example prevents.
- Ask what would need to change if the feature became async, generic, public, or shared across modules.
- Rewrite the example with one deliberate bug, then explain how a reviewer should catch it.
- Turn the example into a two-minute interview explanation with tradeoffs.

