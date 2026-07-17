# Day 5: Collection Performance Basics

## Core Idea

Different collections have different performance characteristics.

| Operation | Array | Set | Dictionary |
| --- | --- | --- | --- |
| Keep order | Yes | No | No guaranteed business order |
| Allow duplicates | Yes | No | Keys no, values yes |
| Lookup by index | Fast | No | No |
| Lookup by key/value | Linear for value | Fast membership | Fast by key |

## Array

Good for ordered data.

```swift
let first = items[0]
```

Searching by value is usually linear.

```swift
items.contains(target)
```

## Set

Good for membership checks.

```swift
selectedIds.contains(id)
```

## Dictionary

Good for lookup by key.

```swift
let user = usersById[id]
```

## Copy-On-Write

Swift collections are value types optimized with copy-on-write.

```swift
var a = [1, 2, 3]
var b = a
b.append(4)
```

Swift avoids copying storage until mutation is needed.

## Modern Swift 6.x Notes

Swift 6.2 added `InlineArray` and `Span` for safe systems programming and fixed/contiguous memory scenarios. They are not replacements for normal collections in everyday iOS UI code, but senior candidates should know they exist.

## Interview Levels

Junior:

Use arrays for lists, sets for unique values, dictionaries for key-value data.

Mid-level:

Choose collections based on lookup, ordering, and uniqueness needs.

Senior:

Performance is about access patterns. Repeated `contains` on a large array may be worse than using a set. Repeated lookup by ID should usually use a dictionary. Avoid premature optimization, but choose the right collection for the data shape.

## Quick Notes

- Array preserves order
- Set gives fast membership
- Dictionary gives fast key lookup
- Swift collections use copy-on-write
- Modern low-level APIs include `InlineArray` and `Span`

## Interview Depth

Junior answer: Different collections are good at different things.

Mid-level answer: Arrays are good for order, sets are good for uniqueness and membership checks, and dictionaries are good for key lookup.

Senior answer: Performance depends on access pattern. A small array is fine for simple searches. But repeated membership checks against thousands of IDs should use a set. Repeated lookup by model ID should use a dictionary. Do not optimize blindly, but choose the structure that matches the operation.

iOS use case:

```swift
let visibleIds: [String] = users.map(\.id)
let usersById = Dictionary(uniqueKeysWithValues: users.map { ($0.id, $0) })
```

Common mistakes:

- Treating Big-O as the only performance factor.
- Optimizing before knowing data size.
- Ignoring sorting cost.
- Forgetting UI updates may dominate collection cost.

Practice:

1. Explain lookup cost for array vs dictionary.
2. Explain why `Set.contains` is useful for selected IDs.
3. Describe copy-on-write.

## Deep Dive Expansion

### Why This Topic Matters In Real Apps

Collection Performance Basics is not only a syntax topic. In production Swift, it affects data shape, performance, identity, ordering, and mutation cost. A junior developer should be able to recognize the syntax and explain the basic behavior. A mid-level developer should know when the feature improves clarity or safety. A senior developer should understand how it changes API design, testability, performance, or long-term maintenance.

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

Use this checklist when applying Collection Performance Basics in an app feature:

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

1. Write a minimal example that shows Collection Performance Basics correctly.
2. Write a second example that demonstrates a common mistake.
3. Refactor the mistake into a safer version.
4. Explain the difference between the beginner solution and the production-ready solution.
5. Add one test case or reasoning check that proves the behavior.

## Applied Day-Level Example

The following example is not the only way to use Collection Performance Basics, but it shows the kind of production shape you should connect this topic to:

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

## Senior iOS Engineer Artifact

### Senior Mental Model

For a senior iOS engineer, the topic **Collection Performance Basics** is evaluated through this lens: collections are data-structure decisions; senior engineers choose them from access pattern, identity, order, and mutation cost. The goal is not to memorize the keyword or syntax in isolation. The goal is to understand how this topic changes correctness, ownership, testability, performance, and team-scale maintainability.

A senior engineer asks:

- What invariant does this protect?
- What future bug does this make harder to introduce?
- What does the call site communicate to another engineer?
- What is the runtime, memory, concurrency, or API-evolution cost?
- Can this design survive a larger feature, a second caller, or a module boundary?

### Artifact To Produce While Studying

Create this artifact for the topic:

```text
Artifact: a collection-choice table listing Array, Set, Dictionary, ordering needs, lookup needs, and expected complexity
Topic: Collection Performance Basics
Owner: Which type/module owns the decision?
Boundary: Is this local, feature-wide, package-wide, public API, actor-isolated, or UI-only?
Invariant: What must always remain true?
Failure Mode: What breaks if this is misused?
Verification: How would I prove it with tests, logs, compiler checks, or tooling?
```

This artifact forces senior-level thinking. It turns a language feature into an engineering decision with evidence.

### Senior-Level Scenario

Imagine building search, favorites, diffable data sources, caches, selected IDs, or grouping data by section. A junior answer usually explains what the syntax does. A senior answer explains the design pressure:

```text
I would first identify the owner and boundary. Then I would choose the smallest surface that preserves the invariant, keeps the call site readable, and avoids leaking implementation details. I would also check the failure path and decide how to verify the behavior with tests or tooling.
```

### What Can Go Wrong

The senior-level risk for this area is using arrays for repeated membership checks or dictionaries where ordering is a product requirement.

That risk matters because iOS apps grow by accumulation. A small unclear decision can become a pattern copied into screens, services, tests, and public APIs.

### Review Checklist

Use this checklist in code review:

- What is the dominant operation?
- Does identity matter?
- Is ordering stable and intentional?
- Is the example still understandable six months later?
- Does the implementation make the safe path easier than the unsafe path?
- If this appears in an interview, can I explain both the syntax and the tradeoff?

### Senior Interview Framing

Use this structure when answering at senior level:

```text
At the syntax level, this feature does X. At the design level, I use it to protect Y. The tradeoff is Z. In a real iOS app, I would apply it in this scenario, verify it this way, and avoid this common misuse.
```

For **Collection Performance Basics**, fill it like this:

```text
At the syntax level, this topic gives me a Swift mechanism for a specific behavior. At the design level, I use it to make ownership, state, or boundaries clearer. The tradeoff is that misuse can hide complexity or create coupling. In a real iOS app, I would apply it where the invariant matters, verify it with focused tests or tooling, and avoid using it just because the syntax is available.
```

