# Day 7: Returning Multiple Values

## Core Idea

Tuples can return multiple values from a function.

```swift
func minMax(_ numbers: [Int]) -> (min: Int, max: Int)? {
    guard let first = numbers.first else { return nil }
    var minValue = first
    var maxValue = first

    for number in numbers {
        minValue = min(minValue, number)
        maxValue = max(maxValue, number)
    }

    return (minValue, maxValue)
}
```

## Tuple Vs Struct

Use tuple for simple local data.

Use struct for domain results.

```swift
struct LoginResult {
    let user: User
    let token: String
}
```

## Interview Levels

Junior:

Tuples can return more than one value.

Senior:

Tuple returns are useful for local algorithms. For public APIs or meaningful domain results, structs scale better.

## Quick Notes

- Tuple return can be named
- Optional tuple can represent no result
- Structs are better for domain data

## Interview Depth

Junior answer: A function can return multiple values using a tuple.

Mid-level answer: Tuple returns are useful for small algorithm results like min/max, coordinates, or split values.

Senior answer: The return type is part of API design. If returned values represent a domain concept, use a struct or enum. Tuples are best for local and simple cases.

iOS use case:

```swift
func visibleRange(page: Int, pageSize: Int) -> (start: Int, end: Int) {
    let start = page * pageSize
    return (start, start + pageSize)
}
```

Common mistakes:

- Returning too many values.
- Using tuple when error modeling is needed.
- Making public APIs harder to evolve.

Practice:

1. Return min/max from numbers.
2. Return `(isValid, message)` and discuss why enum may be better.
3. Explain tuple return vs custom type.

## Deep Dive Expansion

### Why This Topic Matters In Real Apps

Returning Multiple Values is not only a syntax topic. In production Swift, it affects temporary grouping, expressive matching, boundary handling, and readable branching. A junior developer should be able to recognize the syntax and explain the basic behavior. A mid-level developer should know when the feature improves clarity or safety. A senior developer should understand how it changes API design, testability, performance, or long-term maintenance.

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

Use this checklist when applying Returning Multiple Values in an app feature:

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

1. Write a minimal example that shows Returning Multiple Values correctly.
2. Write a second example that demonstrates a common mistake.
3. Refactor the mistake into a safer version.
4. Explain the difference between the beginner solution and the production-ready solution.
5. Add one test case or reasoning check that proves the behavior.

## Applied Day-Level Example

The following example is not the only way to use Returning Multiple Values, but it shows the kind of production shape you should connect this topic to:

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

