# Day 1: Basic Data Types And Literals

## Core Data Types

Swift's most common beginner-level types are:

- `Int`
- `Double`
- `Float`
- `Bool`
- `String`
- `Character`
- `Optional`

## Int

`Int` stores whole numbers.

```swift
let age = 26
let unreadCount = 12
let temperature = -3
```

Use `Int` for counts, indexes, attempts, quantities, and whole-number values.

```swift
var cartItemCount = 0
cartItemCount += 1
```

## Double And Float

`Double` stores decimal numbers with higher precision than `Float`.

```swift
let rating = 4.7
let distanceInKm: Double = 12.5
let animationProgress: Float = 0.75
```

In iOS app code, `Double` is usually the default choice unless an Apple API requires `CGFloat` or `Float`.

For money, prefer `Decimal`:

```swift
let amount: Decimal = 1499.99
```

## Bool

`Bool` stores `true` or `false`.

```swift
let isLoggedIn = true
var isLoading = false
var hasInternetConnection = true
```

Use boolean names that read like yes/no questions.

Good:

```swift
let isPremiumUser = true
let hasCompletedOnboarding = false
let canRetryPayment = true
```

Avoid:

```swift
let premium = true
let onboarding = false
```

## String

`String` stores text.

```swift
let name = "Ananya"
let message = "Welcome, \(name)"
```

String interpolation inserts values into text:

```swift
let score = 95
let result = "Your score is \(score)"
```

Multiline strings:

```swift
let emailBody = """
Hi,

Your interview preparation plan is ready.

Thanks
"""
```

## Character

`Character` stores a single user-perceived character.

```swift
let grade: Character = "A"
let symbol: Character = "?"
```

Swift strings are Unicode-aware, so a character may use more than one underlying Unicode scalar.

## Optional

An optional means a value may be present or absent.

```swift
var selectedUserName: String?
selectedUserName = "Isha"
selectedUserName = nil
```

Use optionals when absence is meaningful:

```swift
struct User {
    let id: Int
    let name: String
    let profileImageURL: URL?
}
```

`profileImageURL` is optional because not every user may have a profile image.

## Literals

A literal is a direct value written in source code.

```swift
let integerLiteral = 100
let floatingLiteral = 99.5
let stringLiteral = "Swift"
let booleanLiteral = true
let arrayLiteral = [1, 2, 3]
let dictionaryLiteral = ["name": "Riya", "city": "Mumbai"]
```

## Numeric Separators

Use underscores to make large numbers readable:

```swift
let population = 1_400_000_000
let fileSize = 10_485_760
```

The underscores do not affect the value.

## Basic Collections Preview

Arrays:

```swift
let topics = ["Variables", "Types", "Operators"]
```

Dictionaries:

```swift
let user = [
    "name": "Rahul",
    "role": "iOS Developer"
]
```

Sets:

```swift
let uniqueSkills: Set<String> = ["Swift", "SwiftUI", "Xcode"]
```

Collections are covered deeply later, but beginners should recognize them early.

## Real iOS Use Cases

### API Model

```swift
struct Product: Decodable {
    let id: Int
    let title: String
    let price: Decimal
    let rating: Double
    let isAvailable: Bool
}
```

### Feature Flag

```swift
let isNewCheckoutEnabled = true
```

### Empty State Message

```swift
let emptyMessage = "No transactions found"
```

### Optional Image URL

```swift
let avatarURL: URL? = URL(string: "https://example.com/avatar.png")
```

## Swift 6.x Relevance

Swift 6.x continues to rely on strong, explicit data modeling. With stricter concurrency checks, simple value types such as `Int`, `String`, `Bool`, and structs containing them are easier to pass across tasks safely when they conform to `Sendable`.

```swift
struct TransactionSummary: Sendable {
    let totalCount: Int
    let totalAmount: Decimal
    let currencyCode: String
}
```

The model is immutable and safe to share as a snapshot.

## Quick Interview Notes

- `Int` is for whole numbers.
- `Double` is the default floating-point type.
- `Float` uses less precision than `Double`.
- `Bool` stores only `true` or `false`.
- `String` stores text.
- `Character` stores a single user-perceived character.
- Optional means a value may be absent.
- Use `Decimal` carefully for money-related values.

## Points To Remember

- Swift is type-safe.
- Choose types that describe the real domain.
- Optional is not an error; it is a model of absence.
- Avoid using `String` for everything.
- Immutable value models are easier to reason about and safer in concurrent code.

## Practice Questions

1. What is the difference between `Double` and `Float`?
2. Why is `Decimal` useful for money?
3. What is an optional?
4. Why should boolean names start with words like `is`, `has`, or `can`?
5. What is a literal?

