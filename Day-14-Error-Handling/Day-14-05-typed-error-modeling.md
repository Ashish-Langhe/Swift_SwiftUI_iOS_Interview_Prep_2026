# Day 14: Typed Error Modeling

## Core Idea

Typed errors model specific failure domains.

```swift
enum NetworkError: Error {
    case invalidURL
    case badStatusCode(Int)
    case decodingFailed
}
```

Swift supports typed throws syntax:

```swift
func makeURL(_ text: String) throws(NetworkError) -> URL {
    guard let url = URL(string: text) else {
        throw .invalidURL
    }
    return url
}
```

## Modern Swift 6.x Notes

Swift compiler docs describe typed throws as `throws(MyError)`. Untyped throws use `any Error`; typed throws can be useful in performance-sensitive or strongly modeled APIs.

## Interview Levels

Junior: Typed errors use enums to describe failure.

Senior: Typed error modeling improves exhaustiveness and caller understanding, but too much specificity can make APIs harder to compose.

## Quick Notes

- Error enums model failure.
- Associated values add context.
- Typed throws can specify thrown error type.

## Interview Depth

Junior answer: Typed error modeling means defining specific error cases with an enum.

Mid-level answer: It makes failure clearer to callers and allows targeted recovery.

Senior answer: Typed throws can make error contracts precise, but highly specific errors can be harder to compose across layers. Map lower-level errors into domain-level errors at boundaries.

iOS use case:

```swift
enum CheckoutError: Error {
    case emptyCart
    case paymentDeclined(reason: String)
}
```

Common mistakes: one giant app error enum, leaking backend details, over-modeling errors with no recovery difference.

Practice: model auth errors, add associated status code, explain typed throws tradeoff.
