# Day 1: Operators, Expressions, And Basic Output

## Core Idea

Operators are symbols or keywords that perform operations on values.

```swift
let total = 10 + 5
let isAdult = age >= 18
let canAccess = isLoggedIn && isPremium
```

An expression is a piece of code that produces a value.

```swift
10 + 5
age >= 18
"Hello, \(name)"
```

## Arithmetic Operators

```swift
let a = 10
let b = 3

let sum = a + b        // 13
let difference = a - b // 7
let product = a * b    // 30
let quotient = a / b   // 3
let remainder = a % b  // 1
```

Integer division removes the decimal part:

```swift
let result = 10 / 3 // 3
```

Use `Double` for decimal division:

```swift
let result = 10.0 / 3.0 // 3.333...
```

## Assignment Operators

```swift
var count = 0
count += 1
count -= 1
count *= 2
count /= 2
```

Swift does not support `count++` or `count--`.

Use:

```swift
count += 1
```

## Comparison Operators

```swift
let isEqual = a == b
let isNotEqual = a != b
let isGreater = a > b
let isLess = a < b
let isGreaterOrEqual = a >= b
let isLessOrEqual = a <= b
```

Comparison operators return `Bool`.

## Logical Operators

```swift
let isLoggedIn = true
let hasSubscription = false

let canWatchPremiumVideo = isLoggedIn && hasSubscription
let canSeePreview = isLoggedIn || hasSubscription
let shouldShowLogin = !isLoggedIn
```

Meanings:

- `&&`: AND
- `||`: OR
- `!`: NOT

## Nil-Coalescing Operator

The nil-coalescing operator provides a fallback for an optional.

```swift
let displayName: String? = nil
let visibleName = displayName ?? "Guest"
```

If `displayName` has a value, `visibleName` uses it. Otherwise, it uses `"Guest"`.

## Range Operators

Closed range:

```swift
for number in 1...5 {
    print(number)
}
```

Half-open range:

```swift
let names = ["A", "B", "C"]

for index in 0..<names.count {
    print(names[index])
}
```

Prefer direct collection iteration when possible:

```swift
for name in names {
    print(name)
}
```

## Ternary Conditional Operator

```swift
let age = 20
let message = age >= 18 ? "Adult" : "Minor"
```

Use it for simple choices. Avoid nesting ternary operators because readability drops quickly.

## String Interpolation

```swift
let name = "Sana"
let completed = 4

let message = "\(name) completed \(completed) lessons."
```

This is common in UI labels, logs, and debug messages.

## Basic Output

Use `print` for simple output:

```swift
print("Hello, Swift")
```

Print multiple values:

```swift
let user = "Dev"
let score = 90

print(user, score)
```

Use interpolation for clearer output:

```swift
print("User: \(user), Score: \(score)")
```

In real iOS apps, prefer logging tools for serious diagnostics, but `print` is fine for beginner examples and quick debugging.

## Comments

Single-line comment:

```swift
// This is a comment
```

Multiline comment:

```swift
/*
 This is a multiline comment.
 It can span multiple lines.
 */
```

Documentation comment:

```swift
/// Returns the final price after applying tax.
func finalPrice(amount: Double, taxRate: Double) -> Double {
    amount + (amount * taxRate)
}
```

Documentation comments are useful for public APIs, utilities, and reusable components.

## Real iOS Use Cases

### Badge Count

```swift
var unreadMessages = 9
unreadMessages += 1
```

### Access Check

```swift
let canOpenSettings = isLoggedIn && !isGuest
```

### Fallback Name

```swift
let title = user.profileName ?? user.email
```

### Price Label

```swift
let quantity = 2
let price = 499.0
let total = Double(quantity) * price

let label = "Total: INR \(total)"
```

## Swift 6.x Relevance

The operators themselves are beginner-level, but Swift 6.x makes expression clarity more important in concurrent code. For example, avoid hiding mutable shared state changes inside complex expressions.

Less clear:

```swift
var count = 0
let message = "Count: \(count += 1)"
```

This style is not valid Swift as written, and even if similar patterns are possible through function calls, they are hard to reason about.

Clearer:

```swift
count += 1
let message = "Count: \(count)"
```

Simple, readable steps make concurrency and debugging easier.

## Quick Interview Notes

- Arithmetic operators include `+`, `-`, `*`, `/`, `%`.
- `==` checks equality; `=` assigns a value.
- `&&`, `||`, and `!` are logical operators.
- `??` gives a default value for an optional.
- `...` creates a closed range; `..<` creates a half-open range.
- Swift removed `++` and `--`; use `+= 1` and `-= 1`.
- `print` is useful for learning but not a complete logging strategy.

## Points To Remember

- Operators should make code shorter, not less readable.
- Be careful with integer division.
- Prefer direct collection iteration over index-based loops when possible.
- Use documentation comments for reusable functions.
- Avoid overly clever expressions in production code.

## Practice Questions

1. What is the difference between `=` and `==`?
2. What does the `%` operator do?
3. What is the difference between `...` and `..<`?
4. Why does `10 / 3` return `3`?
5. What does the nil-coalescing operator do?
