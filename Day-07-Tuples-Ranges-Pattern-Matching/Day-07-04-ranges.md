# Day 7: Ranges

## Core Idea

Ranges represent intervals.

```swift
let closed = 1...5
let halfOpen = 1..<5
```

Closed range includes the end. Half-open range excludes the end.

## Real iOS Use Cases

- Score grading
- Status code handling
- Array slicing
- Validation

```swift
switch statusCode {
case 200..<300:
    print("Success")
case 400..<500:
    print("Client error")
default:
    print("Other")
}
```

## Interview Levels

Junior:

`...` includes the end, `..<` excludes the end.

Senior:

Half-open ranges fit zero-based indexing and slicing. Ranges are also expressive in pattern matching.

## Quick Notes

- `...` closed
- `..<` half-open
- Useful in loops and switch
- Avoid out-of-bounds slicing

## Interview Depth

Junior answer: A range represents values between a start and an end.

Mid-level answer: Closed ranges include the end; half-open ranges exclude the end. Half-open ranges work well with zero-based indexes.

Senior answer: Ranges make classification logic readable, such as status codes, age brackets, pagination, validation, and scoring. Be careful with collection bounds when using ranges for slicing.

iOS use case:

```swift
func message(for statusCode: Int) -> String {
    switch statusCode {
    case 200..<300: return "Success"
    case 400..<500: return "Client error"
    case 500..<600: return "Server error"
    default: return "Unknown"
    }
}
```

Common mistakes:

- Confusing `...` and `..<`.
- Creating invalid ranges.
- Slicing arrays with unchecked bounds.

Practice:

1. Classify scores with ranges.
2. Explain closed vs half-open.
3. Use range in a `switch`.
