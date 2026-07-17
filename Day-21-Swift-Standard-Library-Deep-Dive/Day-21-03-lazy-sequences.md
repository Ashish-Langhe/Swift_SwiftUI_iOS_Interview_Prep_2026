# Day 21: Lazy Sequences

## What Lazy Means

Lazy sequence operations delay work until values are actually consumed.

```swift
let result = numbers.lazy
    .filter { $0.isMultiple(of: 2) }
    .map { $0 * 10 }
    .prefix(3)
```

Without `lazy`, `filter` and `map` usually create intermediate arrays.

With `lazy`, values are processed as needed.

## Why Laziness Matters

Laziness can reduce:

- Intermediate allocations
- Unnecessary work
- Processing time for early-exit operations

Example:

```swift
let firstMatch = users.lazy
    .filter { $0.isActive }
    .map { $0.name }
    .first { $0.hasPrefix("A") }
```

Swift does not need to build arrays for every active name before finding the first match.

## Real iOS Use Case

Search over a large local list:

```swift
func firstVisibleProduct(matching query: String, in products: [Product]) -> Product? {
    products.lazy
        .filter { $0.isAvailable }
        .first { $0.title.localizedCaseInsensitiveContains(query) }
}
```

This is useful when the first result is enough.

## Lazy Is Not Always Faster

Lazy code can be harder to read and may add overhead for small collections.

Use it when:

- Data is large.
- You stop early.
- You want to avoid intermediate arrays.
- Transformations are expensive.

Avoid it when:

- The list is tiny.
- You need all results anyway.
- Readability gets worse.

## Lazy And SwiftUI

Do not confuse standard-library lazy sequences with SwiftUI lazy containers like `LazyVStack`.

```swift
let names = users.lazy.map(\\.name)
```

This is sequence laziness.

```swift
LazyVStack { ... }
```

This is view creation/layout laziness.

Different concepts, similar performance motivation.

## Senior iOS Engineer Artifact

```text
Artifact: Lazy Evaluation Decision
Topic: Lazy Sequences
Input size: small / medium / large
Does operation short-circuit? yes/no
Intermediate allocation avoided: yes/no
Readability cost: low/medium/high
Measured benefit: benchmark, Instruments, or code reasoning
```

Senior lens:

- Use laziness for a reason, not as decoration.
- Prefer clear eager code until allocation or early-exit cost matters.
- Measure performance-sensitive paths.
- Be careful when returning lazy sequences from APIs because evaluation timing moves to the caller.

## More Coding Examples

### Example 1: First Matching User

```swift
let firstAdminName = users.lazy
    .filter { $0.role == .admin }
    .map(\\.name)
    .first
```

### Example 2: Avoid Work After Prefix

```swift
let previewTitles = articles.lazy
    .filter { !$0.isArchived }
    .prefix(5)
    .map(\\.title)
```

Only enough elements are evaluated to produce five visible titles.

## Common Mistakes

- Assuming lazy always improves performance.
- Returning lazy chains where callers expect immediate work.
- Making simple code harder to understand.
- Forgetting side effects inside lazy closures happen later.
- Using lazy to hide inefficient algorithms.

## Interview Guide

Junior:

Lazy means work is delayed until values are needed.

Mid-level:

Lazy sequences help avoid intermediate arrays and unnecessary work, especially when processing large data or stopping early.

Senior:

Lazy evaluation is a performance tool with readability and timing tradeoffs. I use it after understanding data size, short-circuit behavior, and allocation cost. I avoid side effects in lazy transformations.

## Practice

1. Compare eager and lazy `filter/map/first`.
2. Explain why side effects inside lazy closures are risky.
3. Use lazy to get the first five visible products.
4. Decide whether lazy helps for a small settings list.
