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
