# Day 33: SOLID iOS Use Cases And Architecture Alignment

## SOLID With MVVM

MVVM and SOLID work well together.

View:

- Renders UI.
- Sends user intents.

ViewModel:

- Owns presentation state.
- Calls use cases/repositories.

Model/Use Case/Repository:

- Owns business and data behavior.

Example:

```swift
struct ProductsScreen: View {
    @State private var viewModel: ProductsViewModel

    var body: some View {
        ProductList(
            state: viewModel.state,
            retry: { Task { await viewModel.load() } }
        )
    }
}
```

SOLID alignment:

- SRP: View renders, ViewModel coordinates, repository fetches.
- DIP: ViewModel depends on repository protocol.
- ISP: ViewModel only receives product capability.

## SOLID With Coordinator

Coordinator owns navigation.

```swift
protocol ProductRouting: AnyObject {
    func showProductDetails(id: Product.ID)
}
```

ViewModel:

```swift
final class ProductsViewModel {
    weak var router: ProductRouting?

    func didSelectProduct(_ id: Product.ID) {
        router?.showProductDetails(id: id)
    }
}
```

SOLID alignment:

- SRP: Coordinator navigates.
- DIP: ViewModel depends on routing abstraction.
- ISP: ViewModel only knows product routing methods.

## SOLID With Repository

```swift
protocol ProductRepository {
    func products() async throws -> [Product]
}
```

Implementations:

```swift
struct NetworkProductRepository: ProductRepository {}
struct CachedProductRepository: ProductRepository {}
struct PreviewProductRepository: ProductRepository {}
```

SOLID alignment:

- OCP: new data source implementation.
- DIP: ViewModel depends on abstraction.
- LSP: all repositories must honor the same behavior contract.

## SOLID With SwiftUI Components

Bad:

```swift
struct AppButton: View {
    let title: String
    let isPrimary: Bool
    let isDestructive: Bool
    let icon: String?
    let loading: Bool
    let size: ButtonSize
    let action: () -> Void
}
```

This can become too broad.

Better:

```swift
struct PrimaryButton: View {
    let title: String
    let isLoading: Bool
    let action: () -> Void
}

struct DestructiveButton: View {
    let title: String
    let action: () -> Void
}
```

SOLID alignment:

- ISP: smaller component APIs.
- SRP: each component has clear purpose.
- OCP: add specialized components without bloating one type.

## SOLID With TCA

TCA naturally supports some SOLID ideas:

- State/action/reducer separation supports SRP.
- Dependencies support DIP.
- Feature composition supports ISP-like boundaries.
- Reducer composition supports extension.

But TCA can still violate SOLID:

- Giant root reducer.
- Huge action enum.
- Dependencies with too many methods.
- Child feature depends on parent details.

## SOLID With VIPER

VIPER is heavily SRP-oriented:

- View renders.
- Presenter coordinates.
- Interactor handles business.
- Router navigates.
- Entity models data.

VIPER can violate SOLID if:

- Presenter does business logic.
- Interactor formats UI.
- Router makes domain decisions.
- Protocols are too broad.
- Boilerplate hides unclear ownership.

## When SOLID Helps Most

High-value use cases:

- Checkout/payment
- Authentication
- Offline sync
- Search and pagination
- Image upload
- Complex forms
- Large shared services
- SDK integrations
- Feature modules owned by teams

Low-value use cases:

- Tiny row view
- Static screen
- One-off prototype
- Simple local-only helper

## Senior Architecture Decision

Use SOLID when it reduces:

- Change risk
- Test setup
- Coupling
- Repeated edits
- Unclear ownership

Avoid SOLID ceremony when it increases:

- Boilerplate
- Indirection
- Cognitive load
- File hopping
- Premature abstraction

## Senior Artifact

```text
Artifact: SOLID Architecture Alignment
Pattern:
SRP boundary:
OCP extension:
LSP contract:
ISP interfaces:
DIP dependencies:
Benefits:
Tradeoffs:
```

## Interview Notes

Junior:

SOLID can be used with common iOS patterns like MVVM and Coordinator.

Mid-level:

MVVM, Repository, DI, and Coordinator often apply SOLID principles naturally.

Senior:

I align SOLID with architecture patterns by assigning responsibilities deliberately, keeping protocols focused, preserving implementation contracts, and injecting effectful dependencies.

## Practice

1. Explain SOLID in MVVM.
2. Find SOLID violations in a VIPER module.
3. Review a TCA dependency client using ISP/DIP.
4. Decide whether a SwiftUI component is too broad.
