# Day 33: Liskov Substitution Principle

## What LSP Means

Liskov Substitution Principle says:

```text
Subtypes or conforming implementations should be usable wherever the base type or protocol is expected without breaking correctness.
```

In Swift, LSP often applies to:

- Protocol conformers
- Subclasses
- Mock/stub implementations
- Repository implementations
- Service adapters
- ViewModel base classes

## Simple Meaning

If code expects:

```swift
ProductRepository
```

then all implementations should behave according to the same contract.

```swift
NetworkProductRepository
CachedProductRepository
FailingProductRepository
PreviewProductRepository
```

Each must respect what callers expect.

## Bad Example: Repository Contract Violation

Protocol:

```swift
protocol ProductRepository {
    func product(id: Product.ID) async throws -> Product
}
```

Caller expects missing product to throw.

Bad implementation:

```swift
struct PreviewProductRepository: ProductRepository {
    func product(id: Product.ID) async throws -> Product {
        Product(id: id, name: "Fake", price: 0)
    }
}
```

This silently creates fake data for any ID. Tests may pass but production behavior differs.

Better:

```swift
struct PreviewProductRepository: ProductRepository {
    let products: [Product.ID: Product]

    func product(id: Product.ID) async throws -> Product {
        guard let product = products[id] else {
            throw ProductError.notFound
        }

        return product
    }
}
```

Conforming implementations should preserve contract semantics.

## LSP With Subclasses

Bad:

```swift
class ImageLoader {
    func load(url: URL) async throws -> UIImage {
        // load image
    }
}

final class CachedImageLoader: ImageLoader {
    override func load(url: URL) async throws -> UIImage {
        if url.absoluteString.hasPrefix("file://") {
            fatalError("Unsupported")
        }

        return try await super.load(url: url)
    }
}
```

If base loader supports file URLs, subclass cannot reject them unexpectedly.

Better:

- Make contract explicit.
- Avoid inheritance if behavior differs.
- Use composition/protocols.

```swift
protocol ImageLoading {
    func load(url: URL) async throws -> UIImage
}
```

## Preconditions And Postconditions

LSP is about behavior contracts.

Subtypes should not:

- Require stricter preconditions.
- Provide weaker postconditions.
- Throw unexpected errors for valid inputs.
- Break invariants.
- Change side-effect semantics unexpectedly.

Example:

```swift
protocol TokenStore {
    func save(_ token: String) throws
    func token() throws -> String
}
```

If one implementation returns empty string instead of throwing missing-token error, callers behave incorrectly.

## LSP And Test Doubles

Mocks/stubs must also respect contracts.

Bad test double:

```swift
struct InstantSuccessPaymentService: PaymentService {
    func charge(_ request: PaymentRequest) async throws -> PaymentResult {
        .success
    }
}
```

If it ignores invalid amounts, tests do not cover real behavior.

Better:

```swift
struct ValidatingPaymentServiceStub: PaymentService {
    func charge(_ request: PaymentRequest) async throws -> PaymentResult {
        guard request.amount > 0 else {
            throw PaymentError.invalidAmount
        }

        return .success
    }
}
```

## LSP In UIKit

Subclassing view controllers can violate LSP quickly.

Bad:

```swift
class BaseFormViewController: UIViewController {
    func submit() {}
}

final class ReadOnlyFormViewController: BaseFormViewController {
    override func submit() {
        fatalError("Read only")
    }
}
```

If read-only forms cannot submit, they should not inherit a contract that promises submission.

Use composition or protocols:

```swift
protocol SubmittableForm {
    func submit()
}
```

## When To Think About LSP

Use LSP review when:

- You introduce a protocol.
- You create stubs/mocks.
- You subclass.
- You replace service implementations.
- You add cache/offline behavior.
- You wrap third-party SDKs.

## Common Mistakes

- Protocol contract is vague.
- Stub behavior differs from production.
- Subclass disables inherited behavior.
- Mock returns impossible values.
- Cache repository returns stale data where fresh was promised.
- Errors are inconsistent across implementations.

## Senior iOS Engineer Artifact

```text
Artifact: LSP Contract Review
Abstraction:
Expected inputs:
Expected outputs:
Error behavior:
Side effects:
Thread/actor guarantees:
Implementations:
Contract violations:
Tests:
```

## Interview Notes

Junior:

LSP means child types should work wherever parent types are expected.

Mid-level:

Protocol implementations should follow the same behavior contract, including errors and side effects.

Senior:

I use LSP to review abstraction contracts. I make expectations explicit, test all implementations against shared behavior, and avoid inheritance when subclasses cannot honor base semantics.

## Practice

1. Find an LSP violation in a repository stub.
2. Replace unsafe inheritance with protocol composition.
3. Define error behavior for a token store.
4. Explain how mocks can violate LSP.
