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

## Deep Dive Expansion

### Why This Topic Matters In Real Apps

async let is not only a syntax topic. In production Swift, it affects async networking, parallelism, task groups, stream bridging, retry, timeout, and actor-isolated services. A junior developer should be able to recognize the syntax and explain the basic behavior. A mid-level developer should know when the feature improves clarity or safety. A senior developer should understand how it changes API design, testability, performance, or long-term maintenance.

In iOS work, this usually appears while API clients, dashboards, image loaders, WebSocket streams, and token refresh coordination. The important question is not "Can I write the syntax?" The important question is "Does this code make invalid states harder, make ownership clearer, and make future changes safer?"

### Beginner To Senior Progression

Beginner level:

- Define the concept in plain language.
- Write the smallest working example.
- Recognize the compiler error when the feature is used incorrectly.
- Explain what happens at the call site.

Mid-level level:

- Choose this feature over a nearby alternative for a clear reason.
- Handle edge cases, nil/error/cancellation/performance concerns where relevant.
- Keep the code readable for the next developer.
- Write tests around the behavior, not just the implementation detail.

Senior level:

- Explain the design tradeoff and the failure mode it prevents.
- Understand how this feature behaves across module, actor, memory, or API boundaries.
- Design examples that scale from a small screen to a larger feature.
- Avoid exposing implementation details as permanent API.

### Production-Style Example Pattern

Use this checklist when applying async let in an app feature:

1. Identify the owner of the data or behavior.
2. Decide whether the value should be mutable, immutable, optional, throwing, async, isolated, or private.
3. Keep the public surface small and intention-revealing.
4. Add one realistic failure path, not only the happy path.
5. Check whether the code is still understandable from the call site.

```swift
struct FeatureState: Equatable {
    var isLoading: Bool
    var message: String?
    var canRetry: Bool
}

func makeInitialState() -> FeatureState {
    FeatureState(isLoading: false, message: nil, canRetry: false)
}
```

This small pattern is intentionally simple: define the state, control mutation through a narrow function, and make the result easy to inspect in tests.

### Edge Cases To Think About

- What happens when the input is empty, nil, duplicated, delayed, or invalid?
- What happens when this code is called repeatedly from a scrolling list or fast-changing UI?
- Does this API expose too much mutable state?
- Does the implementation assume a specific ordering, lifetime, actor, or thread?
- Will this still be easy to test after the feature grows?

### Topic-Specific Senior Notes

- Always separate task lifetime from object lifetime; a task can outlive the screen that started it.
- State crossing a concurrency boundary should be immutable, actor-isolated, or clearly Sendable.

### Common Interview Follow-Ups

Be ready for these follow-ups:

- Why did you choose this approach instead of the simpler alternative?
- What bug could happen if this is implemented carelessly?
- How would this behave in a large codebase with multiple modules?
- How would you test this without relying on UI screenshots?
- What changes when this code becomes async, public, generic, or shared?

### Strong Interview Framing

A strong answer should sound like this:

```text
I understand the basic syntax, but I also think about ownership and boundaries. I choose the approach that keeps state valid, makes the call site clear, and avoids unnecessary coupling. In a production iOS app, I would also consider testing, cancellation or error behavior where relevant, and whether this should remain an implementation detail or become part of a public API.
```

### Extra Practice

1. Write a minimal example that shows async let correctly.
2. Write a second example that demonstrates a common mistake.
3. Refactor the mistake into a safer version.
4. Explain the difference between the beginner solution and the production-ready solution.
5. Add one test case or reasoning check that proves the behavior.

## Applied Day-Level Example

The following example is not the only way to use async let, but it shows the kind of production shape you should connect this topic to:

```swift
actor ImagePipeline {
    private var inFlight: [URL: Task<UIImage, Error>] = [:]

    func image(for url: URL) async throws -> UIImage {
        if let task = inFlight[url] {
            return try await task.value
        }

        let task = Task { try await downloadAndDecode(url) }
        inFlight[url] = task
        defer { inFlight[url] = nil }
        return try await task.value
    }
}
```

Advanced networking is not only making requests. It is also deduplicating work, respecting cancellation, protecting shared state, and keeping UI updates outside the transport layer.

### How To Study This Example

- Identify which part is syntax and which part is design.
- Ask what invalid state the example prevents.
- Ask what would need to change if the feature became async, generic, public, or shared across modules.
- Rewrite the example with one deliberate bug, then explain how a reviewer should catch it.
- Turn the example into a two-minute interview explanation with tradeoffs.

