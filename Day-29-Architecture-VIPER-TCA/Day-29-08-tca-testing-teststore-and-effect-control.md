# Day 29: TCA Testing, TestStore, And Effect Control

## Why TCA Testing Is Powerful

TCA's biggest strength is exhaustive testing of state and effects.

With `TestStore`, you can:

- Send actions.
- Assert state mutations.
- Receive effect actions.
- Override dependencies.
- Control clocks.
- Test cancellation.
- Test navigation.

## Basic Test

```swift
@Test
func incrementAndDecrement() async {
    let store = TestStore(initialState: CounterFeature.State()) {
        CounterFeature()
    }

    await store.send(.incrementTapped) {
        $0.count = 1
    }

    await store.send(.decrementTapped) {
        $0.count = 0
    }
}
```

## Testing Effects

```swift
@Test
func factButtonLoadsFact() async {
    let store = TestStore(initialState: CounterFeature.State(count: 42)) {
        CounterFeature()
    } withDependencies: {
        $0.numberFactClient.fetch = { "\($0) is a good number." }
    }

    await store.send(.factTapped) {
        $0.isLoading = true
    }

    await store.receive(\.factResponse.success) {
        $0.isLoading = false
        $0.fact = "42 is a good number."
    }
}
```

Exact action matching syntax can vary by TCA version. The core idea is stable: assert sent actions, received effect actions, and state changes.

## Testing Failure

```swift
@Test
func factFailureShowsError() async {
    let store = TestStore(initialState: CounterFeature.State(count: 42)) {
        CounterFeature()
    } withDependencies: {
        $0.numberFactClient.fetch = { _ in throw NetworkError.offline }
    }

    await store.send(.factTapped) {
        $0.isLoading = true
    }

    await store.receive(\.factResponse.failure) {
        $0.isLoading = false
        $0.fact = "Unable to load fact."
    }
}
```

## Testing Debounce With Clock

```swift
@Test
func searchDebounces() async {
    let clock = TestClock()

    let store = TestStore(initialState: SearchFeature.State()) {
        SearchFeature()
    } withDependencies: {
        $0.continuousClock = clock
        $0.searchClient.search = { query in ["Result for \(query)"] }
    }

    await store.send(.queryChanged("s")) {
        $0.query = "s"
    }

    await store.send(.queryChanged("sw")) {
        $0.query = "sw"
    }

    await clock.advance(by: .milliseconds(300))

    await store.receive(\.searchResponse.success) {
        $0.results = ["Result for sw"]
    }
}
```

This proves cancellation/debounce behavior.

## Testing Navigation

```swift
@Test
func tappingProductPushesDetails() async {
    let product = Product.sample
    let store = TestStore(initialState: ProductsFeature.State(products: [product])) {
        ProductsFeature()
    }

    await store.send(.productTapped(product.id)) {
        $0.path.append(.details(ProductDetailsFeature.State(productID: product.id)))
    }
}
```

Navigation is testable because it is state.

## What TCA Tests Force You To Prove

TCA tests are intentionally strict:

- If state changes, assert it.
- If effect emits an action, receive it.
- If dependency matters, override it.
- If time matters, control it.

This can feel heavy, but it catches hidden behavior.

## Common Mistakes

- Not testing effects.
- Using live dependencies.
- Ignoring cancellation.
- Writing overly broad state.
- Skipping navigation tests.
- Fighting TestStore instead of improving reducer design.

## Senior Artifact

```text
Artifact: TCA Test Plan
Feature:
User actions:
State assertions:
Effect responses:
Dependency overrides:
Clock control:
Cancellation:
Navigation:
Failure paths:
Integration flow:
```

## Interview Notes

Junior:

TCA tests send actions and check state.

Mid-level:

`TestStore` also checks effects and received actions.

Senior:

I use TestStore to prove complete feature flows, including dependencies, time, cancellation, navigation, and failure paths. TCA's testing strictness is a design advantage when complexity is high.

## Practice

1. Test increment/decrement.
2. Test a network response effect.
3. Test a failure response.
4. Test debounced search with a test clock.
