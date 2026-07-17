# Day 29: TCA Dependencies, Clients, Live, Preview, And Test Values

## Why Dependencies Are Central In TCA

TCA treats dependencies as first-class architecture.

Dependencies include:

- API clients
- Database clients
- Analytics
- Clock/time
- UUID generation
- Date generation
- Notification center
- Location
- File storage

Reducers should not create live dependencies directly.

## Client Type

```swift
struct ProductClient {
    var products: @Sendable () async throws -> [Product]
    var product: @Sendable (Product.ID) async throws -> Product
}
```

This is a lightweight dependency client.

## Dependency Key

```swift
extension ProductClient: DependencyKey {
    static let liveValue = ProductClient(
        products: {
            try await LiveProductAPI().products()
        },
        product: { id in
            try await LiveProductAPI().product(id: id)
        }
    )
}

extension DependencyValues {
    var productClient: ProductClient {
        get { self[ProductClient.self] }
        set { self[ProductClient.self] = newValue }
    }
}
```

## Reducer Usage

```swift
@Dependency(\.productClient) var productClient

case .task:
    state.isLoading = true
    return .run { send in
        await send(.productsResponse(
            Result { try await productClient.products() }
        ))
    }
```

## Preview Values

Previews should not hit real network.

```swift
extension ProductClient: TestDependencyKey {
    static let previewValue = ProductClient(
        products: { [.sample, .longName] },
        product: { _ in .sample }
    )

    static let testValue = ProductClient(
        products: { unimplemented("ProductClient.products") },
        product: { _ in unimplemented("ProductClient.product") }
    )
}
```

View preview:

```swift
#Preview {
    ProductsView(
        store: Store(initialState: ProductsFeature.State()) {
            ProductsFeature()
        } withDependencies: {
            $0.productClient.products = { [.sample] }
        }
    )
}
```

## Test Overrides

```swift
let store = TestStore(initialState: ProductsFeature.State()) {
    ProductsFeature()
} withDependencies: {
    $0.productClient.products = { [.sample] }
}
```

This keeps tests deterministic.

## Clock Dependency

Time should be injectable.

```swift
@Dependency(\.continuousClock) var clock
```

Use in reducer:

```swift
try await clock.sleep(for: .milliseconds(300))
```

Test:

```swift
let clock = TestClock()

let store = TestStore(initialState: SearchFeature.State()) {
    SearchFeature()
} withDependencies: {
    $0.continuousClock = clock
}
```

## UUID And Date

Inject generated values when tests care.

```swift
@Dependency(\.uuid) var uuid
@Dependency(\.date.now) var now
```

This avoids flaky tests.

## Dependency Design Tips

Good clients:

- Small API surface
- Sendable closures where needed
- Domain-friendly values
- Separate live implementation
- Test defaults fail loudly

Avoid:

- Giant `APIClient` with every endpoint
- Live networking in tests
- Dependencies that expose UIKit
- Hidden singletons inside clients

## Common Mistakes

- Reducer creates `URLSession` or service directly.
- Tests accidentally use live dependency.
- Preview hits network.
- Client has too many unrelated methods.
- Time and UUID not controlled in tests.
- Dependencies return DTOs directly to UI features.

## Senior Artifact

```text
Artifact: TCA Dependency Review
Feature:
Clients:
Live values:
Preview values:
Test values:
Clock/date/uuid:
Sendability:
Failure behavior:
Test overrides:
Client size:
```

## Interview Notes

Junior:

Dependencies are services a feature uses.

Mid-level:

TCA dependencies can be overridden in tests and previews.

Senior:

I design dependency clients as small, testable boundaries with live, preview, and test values. I inject time, UUIDs, and external effects so reducers remain deterministic and testable.

## Practice

1. Create a `ProductClient`.
2. Add live, preview, and test values.
3. Override dependency in `TestStore`.
4. Explain why clocks should be dependencies.
