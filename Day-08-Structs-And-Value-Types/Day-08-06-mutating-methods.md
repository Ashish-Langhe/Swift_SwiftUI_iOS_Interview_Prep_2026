# Day 8: Mutating Methods

## Core Idea

Struct methods cannot change properties unless marked `mutating`.

```swift
struct Counter {
    var value = 0

    mutating func increment() {
        value += 1
    }
}
```

The instance must be `var`.

```swift
var counter = Counter()
counter.increment()
```

## Real iOS Use Cases

- Updating draft models
- Toggling selection
- Changing local state

```swift
struct Selection {
    var ids: Set<String> = []

    mutating func toggle(_ id: String) {
        if ids.contains(id) {
            ids.remove(id)
        } else {
            ids.insert(id)
        }
    }
}
```

## Interview Levels

Junior:

Use `mutating` when a struct method changes its properties.

Senior:

Mutating methods are explicit value changes. They are useful for local state, but returning modified copies can sometimes be easier for functional composition.

## Quick Notes

- Required to modify struct properties
- Instance must be `var`
- Shows mutation clearly
- Useful for value-state updates

## Interview Depth

Junior answer: `mutating` lets a struct method change its properties.

Mid-level answer: Struct methods are non-mutating by default. If a method changes state, Swift requires `mutating` so the mutation is explicit.

Senior answer: Mutating methods are clear for local value updates, but returning a new modified value can be better for immutable pipelines, reducers, and predictable state transitions.

iOS use case:

```swift
struct FormDraft {
    var name = ""

    mutating func normalize() {
        name = name.trimmingCharacters(in: .whitespacesAndNewlines)
    }
}
```

Common mistakes:

- Forgetting `mutating`.
- Calling mutating method on `let`.
- Mutating too much hidden state.

Practice:

1. Write a `toggle` mutating method.
2. Try calling it on `let`.
3. Compare mutating method vs returning copy.

## Deep Dive Expansion

### Why This Topic Matters In Real Apps

Mutating Methods is not only a syntax topic. In production Swift, it affects value semantics, invariants, copying behavior, and predictable state changes. A junior developer should be able to recognize the syntax and explain the basic behavior. A mid-level developer should know when the feature improves clarity or safety. A senior developer should understand how it changes API design, testability, performance, or long-term maintenance.

In iOS work, this usually appears while view state, request models, domain entities, and SwiftUI state values. The important question is not "Can I write the syntax?" The important question is "Does this code make invalid states harder, make ownership clearer, and make future changes safer?"

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

Use this checklist when applying Mutating Methods in an app feature:

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

1. Write a minimal example that shows Mutating Methods correctly.
2. Write a second example that demonstrates a common mistake.
3. Refactor the mistake into a safer version.
4. Explain the difference between the beginner solution and the production-ready solution.
5. Add one test case or reasoning check that proves the behavior.

## Applied Day-Level Example

The following example is not the only way to use Mutating Methods, but it shows the kind of production shape you should connect this topic to:

```swift
struct SearchState: Equatable {
    var query: String = ""
    var results: [SearchResult] = []
    var isLoading = false

    mutating func startSearch(query: String) {
        self.query = query
        self.isLoading = true
        self.results = []
    }
}
```

Value types are ideal for UI state because each mutation creates a clear before/after model that is easy to compare, test, and reason about.

### How To Study This Example

- Identify which part is syntax and which part is design.
- Ask what invalid state the example prevents.
- Ask what would need to change if the feature became async, generic, public, or shared across modules.
- Rewrite the example with one deliberate bug, then explain how a reviewer should catch it.
- Turn the example into a two-minute interview explanation with tradeoffs.

