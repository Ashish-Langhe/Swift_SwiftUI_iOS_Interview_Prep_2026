# Day 20: `async let`

## What `async let` Is

`async let` starts a child task for a value that will be awaited later.

```swift
async let user = api.user()
async let orders = api.orders()

let summary = try await Summary(user: user, orders: orders)
```

It is structured concurrency for a fixed number of child operations.

## When To Use

Use `async let` when:

- You have a small known number of independent operations
- All results are needed
- The work belongs to the current function
- You want automatic cancellation with parent scope

## Sequential vs Parallel

Sequential:

```swift
let a = try await loadA()
let b = try await loadB()
```

Parallel:

```swift
async let a = loadA()
async let b = loadB()
let result = try await combine(a, b)
```

## Scope

`async let` values must be awaited before leaving scope.

```swift
func load() async throws {
    async let value = service.value()
    print(try await value)
}
```

Swift manages child task lifetime.

## Error And Cancellation

If an async-let child throws, awaiting it throws.

If the parent is cancelled, async-let children are cancelled.

## Real iOS Example

```swift
func productDetail(id: Product.ID) async throws -> ProductDetailData {
    async let product = productService.product(id: id)
    async let reviews = reviewService.reviews(productID: id)
    async let recommendations = recommendationService.related(productID: id)

    return try await ProductDetailData(
        product: product,
        reviews: reviews,
        recommendations: recommendations
    )
}
```

## Common Mistakes

- Using `async let` for dependent calls
- Forgetting errors can come from any child
- Starting too many `async let`s manually
- Capturing non-sendable mutable references
- Using it when task group would be clearer

## Modern Swift 6.x Notes

Swift 6 strict concurrency can flag unsafe captures in async-let child work. Prefer immutable values and sendable models.

## Interview Answer

`async let` is best for a small fixed set of independent async operations. It starts child tasks immediately, keeps them scoped to the parent function, and integrates with cancellation and error propagation.

## Practice

1. Use `async let` for profile and settings loading.
2. Explain why dependent network calls should stay sequential.
3. Compare `async let` and task groups.
4. Handle cancellation in an async-let screen load.
