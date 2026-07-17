# Day 30: Factory, Builder, And Assembly Patterns

## Why Creation Patterns Matter

Object creation becomes complicated when screens need:

- ViewModels
- Services
- Repositories
- Routers
- Feature flags
- Environment values
- Test doubles

Creation logic should not leak everywhere.

## Factory Pattern

A factory creates objects.

```swift
protocol ProductsViewModelFactory {
    func makeProductsViewModel() -> ProductsViewModel
}
```

Implementation:

```swift
struct LiveProductsViewModelFactory: ProductsViewModelFactory {
    let dependencies: AppDependencies

    func makeProductsViewModel() -> ProductsViewModel {
        ProductsViewModel(
            repository: dependencies.productRepository,
            analytics: dependencies.analytics
        )
    }
}
```

## Assembly Pattern

Assembly wires a feature module.

```swift
enum ProductsAssembly {
    static func build(dependencies: AppDependencies) -> ProductsViewController {
        let viewModel = ProductsViewModel(
            repository: dependencies.productRepository
        )
        let viewController = ProductsViewController(viewModel: viewModel)
        return viewController
    }
}
```

Common in UIKit, VIPER, and modular apps.

## Builder Pattern

Builder helps construct complex values step by step.

```swift
struct CheckoutRequestBuilder {
    private var cartID: Cart.ID?
    private var address: Address?
    private var paymentMethod: PaymentMethod?

    mutating func setCartID(_ id: Cart.ID) {
        cartID = id
    }

    mutating func setAddress(_ address: Address) {
        self.address = address
    }

    mutating func setPaymentMethod(_ method: PaymentMethod) {
        paymentMethod = method
    }

    func build() throws -> CheckoutRequest {
        guard let cartID, let address, let paymentMethod else {
            throw CheckoutRequestError.missingRequiredFields
        }

        return CheckoutRequest(
            cartID: cartID,
            address: address,
            paymentMethod: paymentMethod
        )
    }
}
```

Use builder when construction has many optional steps or validation.

## Factory vs Assembly vs Builder

Factory:

- Creates one object or family of objects.

Assembly:

- Wires a feature/module graph.

Builder:

- Step-by-step construction of a complex value.

## SwiftUI Factory Example

```swift
struct FeatureFactory {
    let dependencies: AppDependencies

    func makeProductsScreen() -> some View {
        ProductsScreen(
            viewModel: ProductsViewModel(
                repository: dependencies.productRepository
            )
        )
    }
}
```

In SwiftUI, avoid over-abstracting simple view creation.

## When To Use

Use factory/assembly when:

- Creation logic repeats.
- Dependencies are many.
- Feature wiring should be centralized.
- Tests need alternate construction.
- UIKit modules need assembly boundaries.

Use builder when:

- Value construction has multiple steps.
- Required fields need validation.
- Call sites become unreadable.

## Common Mistakes

- Factories that hide global singletons.
- Assembly doing business logic.
- Builder for simple structs.
- Too many factories for tiny objects.
- Construction spread across views/controllers.
- Factory returns overly broad types unnecessarily.

## Senior Artifact

```text
Artifact: Creation Pattern Review
Object/module:
Creation complexity:
Factory needed:
Assembly needed:
Builder needed:
Dependencies:
Validation:
Test construction:
Overengineering risk:
```

## Interview Notes

Junior:

Factory creates objects. Builder constructs complex objects step by step.

Mid-level:

Assembly wires modules and dependencies.

Senior:

I centralize creation when dependency graphs become meaningful, but avoid factories/builders that only hide simple initializers. Assembly should wire, not behave.

## Practice

1. Create an assembly for a UIKit feature.
2. Add a factory for ViewModels.
3. Build a checkout request builder.
4. Explain factory vs builder.
