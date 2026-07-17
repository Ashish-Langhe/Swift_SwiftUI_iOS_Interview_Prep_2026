# Day 15: Autoclosures

## Core Idea

`@autoclosure` automatically wraps an expression in a closure.

```swift
func logIfTrue(_ condition: @autoclosure () -> Bool) {
    if condition() {
        print("True")
    }
}

logIfTrue(2 > 1)
```

## Real Use Cases

- Assertions
- Lazy evaluation
- Custom logging

## Interview Levels

Junior: Autoclosure turns an expression into a closure automatically.

Senior: Autoclosures improve call-site readability but can hide delayed execution. Use sparingly.

## Quick Notes

- Uses `@autoclosure`.
- Delays expression evaluation.
- Can reduce visible closure syntax.

## Interview Depth

Junior answer: `@autoclosure` automatically wraps an expression in a closure.

Mid-level answer: It is used for APIs like assertions where the expression should be evaluated lazily.

Senior answer: Autoclosures can make APIs elegant but hide delayed execution. Avoid them for complex operations or side-effect-heavy expressions.

iOS use case:

```swift
func debugLog(_ message: @autoclosure () -> String) {
    #if DEBUG
    print(message())
    #endif
}
```

Common mistakes: hiding side effects, overusing for normal callbacks, making evaluation timing unclear.

Practice: create debug log autoclosure, explain lazy evaluation, identify bad autoclosure use.
