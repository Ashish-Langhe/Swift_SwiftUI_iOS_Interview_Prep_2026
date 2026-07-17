# Day 5: Dictionaries

## Core Idea

A `Dictionary` stores key-value pairs.

```swift
var scores: [String: Int] = [
    "Aarav": 90,
    "Meera": 85
]
```

Keys must be unique and conform to `Hashable`.

## Common Operations

```swift
scores["Kabir"] = 92
scores["Meera"] = 88
scores["Aarav"] = nil

let meeraScore = scores["Meera"]
```

Dictionary lookup returns an optional because the key may not exist.

```swift
if let score = scores["Meera"] {
    print(score)
}
```

## Real iOS Use Cases

- Cache by ID
- Lookup table
- API JSON-like payloads
- Grouped data
- Feature flags

```swift
let usersById: [String: User] = [
    "u1": User(name: "Aarav"),
    "u2": User(name: "Meera")
]
```

## Dictionary Iteration

```swift
for (name, score) in scores {
    print("\(name): \(score)")
}
```

Do not depend on dictionary order for business logic. Sort when order matters.

```swift
for key in scores.keys.sorted() {
    print(key)
}
```

## Modern Swift 6.x Notes

The basic `Dictionary` API has not changed dramatically in Swift 6.3, but Swift 6's concurrency direction makes immutable dictionary snapshots useful when passing data across tasks. Use value types and `Sendable`-friendly models where possible.

## Interview Levels

Junior:

A dictionary stores values using keys.

Mid-level:

Dictionaries are useful for fast lookup by key. Access returns an optional because the key may be missing.

Senior:

Dictionaries are ideal for identity-based lookup, caches, and grouping. I avoid using them when stable order matters unless I explicitly sort or maintain a separate ordered structure.

## Quick Notes

- Key-value collection
- Keys are unique
- Key must be `Hashable`
- Lookup returns optional
- Great for fast lookup

## Interview Depth

Junior answer: A dictionary stores values using keys. You use the key to get the value.

Mid-level answer: Dictionary lookup returns an optional because a key may not exist. Dictionaries are useful for caches, lookup tables, grouped API data, and normalized state.

Senior answer: Dictionaries are about access patterns. If a screen frequently needs `User` by `id`, storing `[String: User]` avoids repeated array searches. When UI order also matters, keep ordered IDs separately from the dictionary. This pattern is common in Redux-style stores, SwiftUI state containers, and offline caches.

iOS use case:

```swift
struct UserStore {
    private var usersById: [String: User] = [:]

    func user(id: String) -> User? {
        usersById[id]
    }
}
```

Common mistakes:

- Assuming dictionary iteration order is business-stable.
- Force-unwrapping dictionary lookups.
- Using `String: Any` instead of typed models.
- Forgetting keys must be unique.

Practice:

1. Convert `[User]` to `[String: User]`.
2. Safely read a missing key.
3. Explain dictionary vs array for user lookup.

## Deep Dive Expansion

### Why This Topic Matters In Real Apps

Dictionaries is not only a syntax topic. In production Swift, it affects data shape, performance, identity, ordering, and mutation cost. A junior developer should be able to recognize the syntax and explain the basic behavior. A mid-level developer should know when the feature improves clarity or safety. A senior developer should understand how it changes API design, testability, performance, or long-term maintenance.

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

Use this checklist when applying Dictionaries in an app feature:

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

1. Write a minimal example that shows Dictionaries correctly.
2. Write a second example that demonstrates a common mistake.
3. Refactor the mistake into a safer version.
4. Explain the difference between the beginner solution and the production-ready solution.
5. Add one test case or reasoning check that proves the behavior.

## Applied Day-Level Example

The following example is not the only way to use Dictionaries, but it shows the kind of production shape you should connect this topic to:

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

