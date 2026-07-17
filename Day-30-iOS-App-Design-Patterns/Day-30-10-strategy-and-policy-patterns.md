# Day 30: Strategy And Policy Patterns

## What Strategy Pattern Means

Strategy lets you swap algorithms behind a common interface.

Example:

```swift
protocol SortingStrategy {
    func sort(_ products: [Product]) -> [Product]
}
```

Strategies:

```swift
struct PriceLowToHighStrategy: SortingStrategy {
    func sort(_ products: [Product]) -> [Product] {
        products.sorted { $0.price < $1.price }
    }
}

struct NameStrategy: SortingStrategy {
    func sort(_ products: [Product]) -> [Product] {
        products.sorted { $0.name < $1.name }
    }
}
```

Use:

```swift
func visibleProducts(
    _ products: [Product],
    strategy: SortingStrategy
) -> [Product] {
    strategy.sort(products)
}
```

## Why Strategy Matters

Strategy avoids hard-coded conditionals when behavior varies.

Bad:

```swift
switch sort {
case .name:
    return products.sorted { $0.name < $1.name }
case .price:
    return products.sorted { $0.price < $1.price }
case .rating:
    return products.sorted { $0.rating > $1.rating }
}
```

This is fine for small cases. Strategy helps when algorithms grow or need injection/testing.

## Policy Pattern

Policy expresses business decisions.

```swift
struct OrderCancellationPolicy {
    func canCancel(_ order: Order) -> Bool {
        switch order.status {
        case .pending, .confirmed:
            return true
        case .shipped, .delivered, .cancelled:
            return false
        }
    }
}
```

Interactor/use case:

```swift
func cancelOrder(id: Order.ID) async throws {
    let order = try await repository.order(id: id)

    guard cancellationPolicy.canCancel(order) else {
        throw OrderError.cannotCancel
    }

    try await repository.cancel(id: id)
}
```

## Strategy With Closures

Swift makes lightweight strategies easy.

```swift
struct ProductSorter {
    let sort: ([Product]) -> [Product]
}
```

Use:

```swift
let byRating = ProductSorter {
    $0.sorted { $0.rating > $1.rating }
}
```

Use protocols when the strategy has meaningful type identity or multiple methods. Use closures when behavior is simple.

## Real iOS Use Cases

Strategy/policy examples:

- Sorting products
- Filtering search results
- Retry policy
- Cache policy
- Pricing rules
- Feature flag evaluation
- Order cancellation
- Password validation
- Routing decision

## Retry Policy Example

```swift
struct RetryPolicy {
    let maxAttempts: Int
    let delay: Duration

    func shouldRetry(error: Error, attempt: Int) -> Bool {
        attempt < maxAttempts && error is NetworkError
    }
}
```

Service:

```swift
func loadWithRetry() async throws -> [Product] {
    var attempt = 0

    while true {
        do {
            return try await api.products()
        } catch {
            guard policy.shouldRetry(error: error, attempt: attempt) else {
                throw error
            }

            attempt += 1
            try await clock.sleep(for: policy.delay)
        }
    }
}
```

## Common Mistakes

- Strategy objects for tiny one-line switches.
- Business policies hidden in views.
- Policies that depend on UIKit.
- Untested policy edge cases.
- Too many tiny classes instead of simple closures/enums.
- Strategy chosen globally when it should be per feature.

## Senior Artifact

```text
Artifact: Strategy/Policy Review
Variable behavior:
Strategies:
Business rules:
Closure vs protocol:
Injection point:
Edge cases:
Tests:
Overengineering risk:
```

## Interview Notes

Junior:

Strategy lets you swap algorithms.

Mid-level:

Policies capture business rules like whether an order can be cancelled.

Senior:

I extract strategies and policies when behavior varies meaningfully or business rules need tests. I keep simple switches when abstraction would add noise.

## Practice

1. Create sorting strategies for products.
2. Extract order cancellation policy.
3. Build a retry policy.
4. Decide closure vs protocol strategy.
