# Day 5: Arrays

## Core Idea

An `Array` stores ordered values of the same type.

```swift
let topics = ["Arrays", "Dictionaries", "Sets"]
var scores = [90, 85, 92]
```

Arrays preserve order and allow duplicate values.

```swift
let numbers = [1, 1, 2, 3]
```

## Common Operations

```swift
var names = ["Aarav", "Meera"]

names.append("Kabir")
names.insert("Riya", at: 1)
names.remove(at: 0)

let first = names.first
let count = names.count
let isEmpty = names.isEmpty
```

Array indexing can crash if the index is invalid.

```swift
if names.indices.contains(1) {
    print(names[1])
}
```

## Real iOS Use Cases

- Table view or SwiftUI list data
- Search results
- Transaction history
- Form fields
- Ordered navigation steps

```swift
struct Transaction {
    let title: String
    let amount: Decimal
}

let transactions = [
    Transaction(title: "Food", amount: 250),
    Transaction(title: "Travel", amount: 1200)
]
```

## Modern Swift 6.x Notes

Swift 6.2 introduced `InlineArray`, a fixed-size array-like storage feature for low-level and performance-sensitive code. Most iOS app code should continue using normal `Array`; mention `InlineArray` in interviews only as a modern systems/performance feature.

## Interview Levels

Junior:

An array stores multiple values in order.

Mid-level:

Arrays are ordered, index-based collections. They are good when order matters or duplicates are allowed.

Senior:

Arrays are excellent for ordered data and UI lists, but membership checks are linear unless additional indexing is used. For uniqueness or frequent lookup, a `Set` or `Dictionary` may be better.

## Quick Notes

- Ordered collection
- Allows duplicates
- Zero-based indexing
- Invalid index access crashes
- Best for ordered UI data

## Interview Depth

Junior answer: An array is an ordered list of values of the same type. You use it when you need to keep items in sequence.

Mid-level answer: Arrays are useful for UI lists, ordered API results, navigation steps, and any data where duplicate values are allowed. Access by index is fast, but searching by value requires scanning.

Senior answer: Arrays are often the right structure for ordered rendering, but they are not always the right structure for lookup. If code repeatedly calls `contains` on a large array, a `Set` may be better. If code repeatedly searches objects by ID, a `Dictionary` may be better. In production, I choose arrays for order and combine them with dictionaries when I need both order and fast lookup.

iOS use case:

```swift
struct FeedViewModel {
    private(set) var posts: [Post] = []

    mutating func replacePosts(_ newPosts: [Post]) {
        posts = newPosts
    }
}
```

Common mistakes:

- Force-indexing without checking bounds.
- Using arrays for uniqueness.
- Using arrays for repeated ID lookup.
- Mutating arrays from multiple concurrent contexts without isolation.

Practice:

1. Build an array of transactions and return only the first three.
2. Safely access the fifth element.
3. Explain why an array is good for a table view data source.

## Deep Dive Expansion

### Why This Topic Matters In Real Apps

Arrays is not only a syntax topic. In production Swift, it affects data shape, performance, identity, ordering, and mutation cost. A junior developer should be able to recognize the syntax and explain the basic behavior. A mid-level developer should know when the feature improves clarity or safety. A senior developer should understand how it changes API design, testability, performance, or long-term maintenance.

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

Use this checklist when applying Arrays in an app feature:

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

1. Write a minimal example that shows Arrays correctly.
2. Write a second example that demonstrates a common mistake.
3. Refactor the mistake into a safer version.
4. Explain the difference between the beginner solution and the production-ready solution.
5. Add one test case or reasoning check that proves the behavior.

## Applied Day-Level Example

The following example is not the only way to use Arrays, but it shows the kind of production shape you should connect this topic to:

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

