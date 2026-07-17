# Day 5: Iteration Patterns

## Core Idea

Iteration means going through collection values.

```swift
for item in items {
    print(item)
}
```

## Common Patterns

Array values:

```swift
for name in names {
    print(name)
}
```

Index and value:

```swift
for (index, name) in names.enumerated() {
    print("\(index): \(name)")
}
```

Dictionary:

```swift
for (id, user) in usersById {
    print(id, user)
}
```

Set:

```swift
for tag in tags.sorted() {
    print(tag)
}
```

## Where Clause

```swift
for user in users where user.isActive {
    print(user.name)
}
```

## Real iOS Use Cases

- Rendering rows
- Validating fields
- Building view models
- Grouping API data

```swift
let rows = users.map { UserRow(title: $0.name) }
```

## Modern Swift 6.x Notes

For async data streams, Swift also has `for await` with `AsyncSequence`. That belongs to concurrency, but interviewers may connect iteration with modern async APIs.

## Interview Levels

Junior:

Use `for-in` to loop over collection items.

Mid-level:

Use `enumerated()` when index is needed, and sort dictionaries or sets when order matters.

Senior:

Choose iteration style based on intent. Use loops for control flow and early exits; use higher-order functions for transformations; avoid depending on unordered collection order.

## Quick Notes

- Use `for-in` for simple loops
- Use `enumerated()` for index and value
- Use `where` for simple filtering
- Sort unordered collections when order matters
- Use loops when you need `break` or `continue`

## Interview Depth

Junior answer: Iteration means going through each item in a collection using a loop.

Mid-level answer: Use direct `for-in` when you only need values, `enumerated()` when you need index and value, and dictionary tuple iteration for key-value pairs.

Senior answer: Iteration style communicates intent. A `for` loop is best for early exit, mutation, and complex branching. Higher-order methods are best for clear transformations. Avoid relying on set or dictionary order unless sorted.

iOS use case:

```swift
for (index, section) in sections.enumerated() {
    print("Render section \(index): \(section.title)")
}
```

Common mistakes:

- Using indices when values are enough.
- Relying on unordered collection order.
- Using `forEach` when `break` is needed.

Practice:

1. Iterate users with index.
2. Print dictionary keys sorted.
3. Explain `forEach` vs `for-in`.

## Deep Dive Expansion

### Why This Topic Matters In Real Apps

Iteration Patterns is not only a syntax topic. In production Swift, it affects data shape, performance, identity, ordering, and mutation cost. A junior developer should be able to recognize the syntax and explain the basic behavior. A mid-level developer should know when the feature improves clarity or safety. A senior developer should understand how it changes API design, testability, performance, or long-term maintenance.

In iOS work, this usually appears while lists, lookup tables, selected IDs, search filtering, and diffable UI data sources. The important question is not "Can I write the syntax?" The important question is "Does this code make invalid states harder, make ownership clearer, and make future changes safer?"

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

Use this checklist when applying Iteration Patterns in an app feature:

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

- Choose collections by access pattern: ordered traversal, uniqueness, membership lookup, or key-based lookup.
- Avoid repeated linear scans in hot UI paths; convert to dictionaries or sets when lookup dominates.

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

1. Write a minimal example that shows Iteration Patterns correctly.
2. Write a second example that demonstrates a common mistake.
3. Refactor the mistake into a safer version.
4. Explain the difference between the beginner solution and the production-ready solution.
5. Add one test case or reasoning check that proves the behavior.

## Applied Day-Level Example

The following example is not the only way to use Iteration Patterns, but it shows the kind of production shape you should connect this topic to:

```swift
struct Contact: Identifiable, Hashable {
    let id: UUID
    let name: String
    let phone: String
}

let contacts: [Contact] = loadContacts()
let contactsByID = Dictionary(uniqueKeysWithValues: contacts.map { ($0.id, $0) })
let favoriteIDs: Set<Contact.ID> = Set(loadFavoriteIDs())

let visibleFavorites = contacts.filter { favoriteIDs.contains($0.id) }
```

This is how collections show up in real apps: arrays preserve display order, dictionaries give direct lookup, and sets make membership checks fast.

### How To Study This Example

- Identify which part is syntax and which part is design.
- Ask what invalid state the example prevents.
- Ask what would need to change if the feature became async, generic, public, or shared across modules.
- Rewrite the example with one deliberate bug, then explain how a reviewer should catch it.
- Turn the example into a two-minute interview explanation with tradeoffs.

