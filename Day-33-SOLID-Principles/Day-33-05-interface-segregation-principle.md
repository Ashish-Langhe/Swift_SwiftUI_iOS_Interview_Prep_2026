# Day 33: Interface Segregation Principle

## What ISP Means

Interface Segregation Principle says:

```text
Clients should not be forced to depend on methods they do not use.
```

In Swift, this usually means:

- Prefer small focused protocols.
- Avoid giant manager protocols.
- Split read/write capabilities.
- Keep feature dependencies minimal.

## Bad Example: Huge Service Protocol

```swift
protocol AppService {
    func login(email: String, password: String) async throws -> User
    func logout() async throws
    func products() async throws -> [Product]
    func orders() async throws -> [Order]
    func track(event: String)
    func saveImage(_ image: UIImage) async throws -> URL
    func refreshToken() async throws
}
```

A `ProductsViewModel` that only needs products now depends on auth, analytics, orders, and image upload.

## Better: Segregated Protocols

```swift
protocol AuthServicing {
    func login(email: String, password: String) async throws -> User
    func logout() async throws
}

protocol ProductServicing {
    func products() async throws -> [Product]
}

protocol AnalyticsTracking {
    func track(event: String)
}

protocol ImageUploading {
    func saveImage(_ image: UIImage) async throws -> URL
}
```

ViewModel depends only on what it uses:

```swift
@Observable
@MainActor
final class ProductsViewModel {
    private let productService: ProductServicing

    init(productService: ProductServicing) {
        self.productService = productService
    }
}
```

## Read/Write Segregation

```swift
protocol UserReading {
    func currentUser() async throws -> User
}

protocol UserWriting {
    func updateUser(_ user: User) async throws
}
```

A profile header only needs reading:

```swift
final class ProfileHeaderViewModel {
    private let userReader: UserReading
}
```

An edit screen needs writing:

```swift
final class EditProfileViewModel {
    private let userReader: UserReading
    private let userWriter: UserWriting
}
```

## ISP In SwiftUI Components

Bad component API:

```swift
struct ProductRow: View {
    let product: Product
    let onTap: () -> Void
    let onDelete: () -> Void
    let onFavorite: () -> Void
    let onShare: () -> Void
    let onCompare: () -> Void
}
```

If most screens do not use all actions, the interface is too broad.

Better:

```swift
struct ProductRow: View {
    let product: Product
    let action: ProductRowAction?
}

struct ProductRowAction {
    let open: (() -> Void)?
    let favorite: (() -> Void)?
}
```

Or split components:

```swift
ProductSummaryRow(product: product)
EditableProductRow(product: product, onDelete: delete)
FavoriteProductRow(product: product, onFavorite: favorite)
```

## ISP And Testing

Smaller protocols make tests simpler.

Bad test setup:

```swift
struct ProductServiceStub: AppService {
    func login(...) async throws -> User { fatalError() }
    func logout() async throws {}
    func products() async throws -> [Product] { [.sample] }
    func orders() async throws -> [Order] { fatalError() }
    func track(event: String) {}
    func saveImage(_ image: UIImage) async throws -> URL { fatalError() }
    func refreshToken() async throws {}
}
```

Better:

```swift
struct ProductServiceStub: ProductServicing {
    func products() async throws -> [Product] {
        [.sample]
    }
}
```

## When To Apply ISP

Apply when:

- Protocol has unrelated methods.
- Test doubles require many unused stubs.
- Type depends on more than it needs.
- Feature imports broad infrastructure.
- Protocol changes break unrelated clients.

Do not over-apply when:

- Protocol is already cohesive.
- Splitting creates confusing tiny protocols.
- Methods are always used together.

## Common Mistakes

- One `APIClientProtocol` with all endpoints.
- One `AppManagerProtocol` for everything.
- Test stubs full of `fatalError`.
- Protocols split so small they lose meaning.
- View components with too many optional callbacks.
- Clients depending on concrete app containers directly.

## Senior iOS Engineer Artifact

```text
Artifact: ISP Review
Client:
Current dependency:
Methods used:
Methods unused:
Suggested smaller protocols:
Test impact:
Change impact:
Tradeoff:
```

## Interview Notes

Junior:

ISP means do not force code to depend on methods it does not need.

Mid-level:

Create smaller protocols like `ProductServicing`, `AnalyticsTracking`, and `ImageUploading`.

Senior:

I use ISP to reduce dependency surface area, test setup, and change blast radius. I split interfaces around cohesive capabilities, not arbitrary one-method protocol noise.

## Practice

1. Split a giant `AppService` protocol.
2. Refactor a test stub with unused methods.
3. Design read/write user protocols.
4. Explain when splitting protocols becomes noise.
