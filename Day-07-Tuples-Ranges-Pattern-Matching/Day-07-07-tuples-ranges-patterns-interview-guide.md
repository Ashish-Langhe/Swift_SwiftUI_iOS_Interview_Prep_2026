# Day 7: Tuples, Ranges, And Pattern Matching Interview Guide

## One-Minute Interview Answer

Tuples group small related values, ranges represent intervals, and pattern matching lets Swift check values by shape. Swift `switch` can match ranges, tuples, enums, optionals, and conditions using `where`. I use tuples for local temporary grouping, structs for domain data, ranges for validation and classification, and pattern matching for enum-driven state handling.

## Modern Swift 6.x Notes

Swift 6.3 improved C interoperability with `@c`, including exposing Swift enums to C. For app interviews, the more important point is still exhaustive Swift enum switching and pattern matching.

## Junior Questions

What is a tuple?

A group of multiple values.

What is a range?

A sequence or interval between values.

## Senior Questions

When avoid tuples?

When the data has domain meaning, needs methods, protocol conformance, documentation, or API stability.

Why is pattern matching powerful?

It makes state handling explicit and compiler-checked.

## Common Traps

- Using tuples for complex domain models
- Forgetting half-open range excludes end
- Overusing `if case` when switch is clearer
- Hiding future enum cases with `default`

## Topic-By-Topic Deep Dive

### Tuples

```swift
let coordinate = (x: 10, y: 20)
```

Tuples are lightweight. They are not meant to become large domain models.

Senior note:

If a tuple appears in many signatures, create a struct. Structs can conform to protocols, have documentation, methods, and stable API meaning.

### Named Tuple Elements

```swift
func screenSize() -> (width: Double, height: Double) {
    (390, 844)
}
```

Names make tuple use readable, especially for return values.

### Returning Multiple Values

Tuple returns are fine for small algorithmic results:

```swift
func bounds(of numbers: [Int]) -> (min: Int, max: Int)?
```

Senior note:

For domain outcomes, use a named type:

```swift
struct LoginSuccess {
    let user: User
    let token: String
}
```

### Ranges

```swift
case 200..<300:
    print("Success")
```

Ranges are expressive for validation, status codes, paging, and scoring.

### Pattern Matching In Switch

Swift switch can match values by structure.

```swift
switch response {
case (200..<300, let data):
    print("Success with \(data.count) bytes")
default:
    print("Failure")
}
```

Senior note:

Pattern matching is not just syntax. It is a way to model state transitions clearly.

### If Case, Guard Case, For Case

```swift
guard case .loaded(let user) = state else {
    return
}
```

Use these when one pattern is important. Use full `switch` when multiple states deserve explicit handling.

## Production Decision Table

| Need | Feature |
| --- | --- |
| Temporary group | Tuple |
| Readable tuple fields | Named tuple |
| Small multi-value algorithm result | Tuple return |
| Domain result | Struct |
| Interval matching | Range |
| Enum state handling | Switch pattern matching |
| One-case extraction | `if case` / `guard case` |

## Final Revision

- Tuples are lightweight groups
- Named tuple elements improve readability
- Ranges classify values
- Switch supports deep pattern matching
- `if case`, `guard case`, `for case` match one pattern

## Deep Dive Expansion

### Why This Topic Matters In Real Apps

Tuples, Ranges, And Pattern Matching Interview Guide is not only a syntax topic. In production Swift, it affects temporary grouping, expressive matching, boundary handling, and readable branching. A junior developer should be able to recognize the syntax and explain the basic behavior. A mid-level developer should know when the feature improves clarity or safety. A senior developer should understand how it changes API design, testability, performance, or long-term maintenance.

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

Use this checklist when applying Tuples, Ranges, And Pattern Matching Interview Guide in an app feature:

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

1. Write a minimal example that shows Tuples, Ranges, And Pattern Matching Interview Guide correctly.
2. Write a second example that demonstrates a common mistake.
3. Refactor the mistake into a safer version.
4. Explain the difference between the beginner solution and the production-ready solution.
5. Turn one answer into a real project story with tradeoffs and verification.

## Applied Day-Level Example

The following example is not the only way to use Tuples, Ranges, And Pattern Matching Interview Guide, but it shows the kind of production shape you should connect this topic to:

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

