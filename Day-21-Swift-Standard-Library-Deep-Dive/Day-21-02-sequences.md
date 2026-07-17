# Day 21: Sequences

## What A Sequence Is

`Sequence` is the simplest standard-library abstraction for values that can be iterated.

```swift
for value in [1, 2, 3] {
    print(value)
}
```

Arrays, sets, dictionaries, ranges, strings, and many custom types are sequences.

## Key Idea

A sequence may be single-pass. That means you should not always assume you can iterate it repeatedly and get the same result.

```swift
func printValues<S: Sequence>(_ values: S) {
    for value in values {
        print(value)
    }
}
```

This function only needs one pass, so `Sequence` is enough.

## Custom Sequence

```swift
struct Countdown: Sequence {
    let start: Int

    func makeIterator() -> CountdownIterator {
        CountdownIterator(current: start)
    }
}

struct CountdownIterator: IteratorProtocol {
    var current: Int

    mutating func next() -> Int? {
        guard current >= 0 else { return nil }
        defer { current -= 1 }
        return current
    }
}

for value in Countdown(start: 3) {
    print(value)
}
```

This teaches the mental model behind `for-in`.

## Sequence Operations

Common sequence methods:

```swift
let names = users.map(\\.name)
let active = users.filter(\\.isActive)
let firstAdmin = users.first { $0.role == .admin }
let total = orders.reduce(0) { $0 + $1.amount }
```

These operations are expressive but should still be readable.

## Real iOS Use Cases

- Transform API DTOs into view models.
- Filter visible rows.
- Find selected items.
- Sum cart totals.
- Build analytics payloads.

```swift
let sections = products
    .filter { $0.isAvailable }
    .map { ProductRowViewModel(product: $0) }
```

## Single-Pass Caution

If your function needs to iterate more than once, use `Collection`.

Bad:

```swift
func hasDuplicates<S: Sequence>(_ values: S) -> Bool where S.Element: Hashable {
    let array = Array(values)
    return Set(array).count != array.count
}
```

This works by materializing the sequence, but the signature hides that cost.

Better:

```swift
func hasDuplicates<C: Collection>(_ values: C) -> Bool where C.Element: Hashable {
    Set(values).count != values.count
}
```

## Senior iOS Engineer Artifact

```text
Artifact: Sequence Usage Review
Topic: Sequences
Operation: map/filter/reduce/custom iteration
Passes needed: one or multiple?
Materialization cost: none / Array conversion / dictionary grouping
Input size: small UI list / large payload / unbounded stream
Readability: chain is clear or should be split?
```

Senior lens:

- Use sequence APIs to express intent.
- Split long chains when debugging or readability suffers.
- Avoid repeated transformations in hot UI paths.
- Know when laziness or a collection constraint is better.

## More Coding Examples

### Example 1: Build Cell Models

```swift
let rows = users.map { user in
    UserRowViewModel(id: user.id, title: user.name)
}
```

### Example 2: Reduce Cart Total

```swift
let total = cartItems.reduce(Decimal.zero) { partial, item in
    partial + item.price
}
```

## Common Mistakes

- Using `reduce` when a loop is clearer.
- Assuming a sequence can be traversed many times.
- Creating large intermediate arrays accidentally.
- Doing expensive transformations inside SwiftUI `body` repeatedly.
- Hiding errors by chaining too much.

## Interview Guide

Junior:

A sequence is something you can iterate with `for-in`.

Mid-level:

Sequence operations like `map`, `filter`, and `reduce` help transform data, but you should understand when they create arrays and when multiple passes are needed.

Senior:

`Sequence` is the minimal iteration abstraction. I use it for one-pass algorithms, but I move to `Collection` when I need count, indexing, or repeated traversal. I also consider laziness and allocation cost in UI-heavy paths.

## Practice

1. Create a custom countdown sequence.
2. Convert a list of users into row models.
3. Explain when `Collection` is better than `Sequence`.
4. Refactor a complex chain into readable steps.
