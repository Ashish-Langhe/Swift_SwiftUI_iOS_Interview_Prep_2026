# Day 7: Tuples

## Core Idea

A tuple groups multiple values into one compound value.

```swift
let point = (10, 20)
print(point.0)
print(point.1)
```

Tuples are lightweight and useful for temporary grouping.

## Real iOS Use Cases

- Returning small pairs
- Coordinates
- Intermediate calculations
- Simple grouped loop values

```swift
let size = (width: 100, height: 200)
```

## Interview Levels

Junior:

A tuple groups values together.

Senior:

Tuples are good for small local groupings, but domain data should usually become a struct for readability, documentation, and evolution.

## Quick Notes

- Lightweight grouping
- Can be named or unnamed
- Good for temporary values
- Prefer structs for domain models

## Interview Depth

Junior answer: A tuple groups multiple values into one value.

Mid-level answer: Tuples are useful for small temporary groupings, especially inside functions or local calculations.

Senior answer: Tuples are not a replacement for domain types. If a tuple has meaning in your app, appears in multiple places, or needs protocol conformance, create a struct.

iOS use case:

```swift
let layout = (columns: 2, spacing: 12.0)
```

Common mistakes:

- Returning large tuples.
- Using `.0`, `.1` in complex code.
- Exposing tuple-heavy public APIs.

Practice:

1. Create a tuple for width/height.
2. Destructure a tuple.
3. Explain tuple vs struct.
