# Day 1: Type Inference And Type Annotations

## Core Idea

Swift is a strongly typed language. Every value has a type. You can either let the compiler infer the type or write the type explicitly.

```swift
let city = "Pune"      // Inferred as String
let year = 2026        // Inferred as Int
let progress = 0.75    // Inferred as Double
let isActive = true    // Inferred as Bool
```

Type annotation means writing the type yourself:

```swift
let city: String = "Pune"
let year: Int = 2026
let progress: Double = 0.75
let isActive: Bool = true
```

## Why Type Inference Matters

Type inference keeps Swift concise without making it dynamically typed.

```swift
let userName = "Meera"
```

The compiler knows `userName` is a `String`. You do not need to write:

```swift
let userName: String = "Meera"
```

Both are valid. The first is more common when the type is obvious.

## When To Use Type Annotation

### 1. Declaring Without Initial Value

```swift
var selectedEmail: String
selectedEmail = "user@example.com"
```

The compiler needs the type because there is no initial value.

### 2. Choosing A Specific Numeric Type

```swift
let count: Int = 10
let price: Double = 99.99
let preciseAmount: Decimal = 199.99
```

For financial values, `Decimal` is often safer than `Double`.

### 3. Improving Readability

```swift
let completionHandler: (Result<String, Error>) -> Void
```

The explicit type helps the reader understand what kind of closure is expected.

### 4. Working With Protocols

```swift
protocol PaymentService {
    func pay(amount: Decimal)
}

let service: PaymentService = StripePaymentService()
```

The annotation hides the concrete type and exposes only the protocol contract.

## Type Safety

Swift does not silently mix unrelated types.

```swift
let age = 30
let name = "Ravi"

// let result = age + name
// Error: Binary operator '+' cannot be applied to operands of type 'Int' and 'String'
```

You must convert intentionally:

```swift
let result = "\(name) is \(age) years old"
```

## Numeric Inference

Integer literals are inferred as `Int` by default:

```swift
let quantity = 5 // Int
```

Floating-point literals are inferred as `Double` by default:

```swift
let discount = 0.10 // Double
```

You can ask for `Float`:

```swift
let animationDuration: Float = 0.35
```

In most app code, `Double` is preferred unless an API specifically requires `Float`.

## Type Conversion

Swift does not automatically convert between numeric types.

```swift
let quantity = 3
let price = 49.99

// let total = quantity * price
// Error: Cannot convert value of type 'Int' to expected argument type 'Double'
```

Correct:

```swift
let total = Double(quantity) * price
```

This explicit conversion avoids hidden precision or overflow bugs.

## Type Aliases

You can create a name for an existing type:

```swift
typealias UserId = String
typealias Completion = (Result<Data, Error>) -> Void

let id: UserId = "user-101"
```

Type aliases improve readability but do not create a new distinct type.

```swift
let userId: UserId = "u-1"
let plainString: String = userId // Allowed
```

## Real iOS Use Cases

### URL Construction

```swift
let baseURL: URL = URL(string: "https://api.example.com")!
```

Explicit `URL` makes the intended type clear.

### View Model State

```swift
final class LoginViewModel {
    var email: String = ""
    var password: String = ""
    var errorMessage: String?
}
```

`String?` clearly communicates that an error may or may not exist.

### API Response

```swift
struct UserResponse: Decodable {
    let id: Int
    let name: String
    let isVerified: Bool
}
```

The model's type annotations document the API contract.

## Swift 6.x Relevance

Modern Swift uses types not just for values, but also for safety across concurrency boundaries.

For example, `Sendable` tells Swift that a value can safely cross concurrency domains:

```swift
struct UserSnapshot: Sendable {
    let id: String
    let name: String
}
```

This matters in Swift 6 because strict concurrency checking catches more unsafe sharing.

## Quick Interview Notes

- Swift is statically and strongly typed.
- Type inference means the compiler determines the type from context.
- Type annotation means the developer writes the type explicitly.
- Integer literals default to `Int`.
- Floating-point literals default to `Double`.
- Swift does not perform implicit numeric conversions.
- Use explicit types when they improve clarity, when no initial value exists, or when a specific API contract matters.

## Points To Remember

- Type inference reduces noise, not type safety.
- Explicit conversions are intentional in Swift.
- Protocol-typed variables can hide implementation details.
- Type aliases improve readability but do not create new types.
- Swift 6 concurrency features rely heavily on precise type information.

## Practice Questions

1. What is type inference?
2. Why does Swift avoid implicit conversion from `Int` to `Double`?
3. When should you prefer explicit type annotations?
4. What is the default type of `10` and `10.0`?
5. Does `typealias UserId = String` create a new type?

