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
