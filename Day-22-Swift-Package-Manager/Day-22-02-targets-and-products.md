# Day 22: Targets And Products

## What Targets Are

A target is a module that Swift builds.

Common target types:

- `.target`
- `.testTarget`
- `.executableTarget`
- `.plugin`
- `.macro`

```swift
targets: [
    .target(name: "NetworkingCore"),
    .testTarget(name: "NetworkingCoreTests", dependencies: ["NetworkingCore"]),
]
```

Each target has source files and dependencies.

## What Products Are

A product is what the package exposes to clients.

```swift
products: [
    .library(name: "NetworkingKit", targets: ["NetworkingCore"]),
]
```

Other packages import products, not every internal target.

## Target vs Product

```text
Target: build unit/module inside the package
Product: public offering exposed to package clients
```

You may have internal targets that are not exposed as products.

```swift
targets: [
    .target(name: "NetworkingCore"),
    .target(name: "NetworkingInternalSupport"),
]

products: [
    .library(name: "NetworkingKit", targets: ["NetworkingCore"]),
]
```

`NetworkingInternalSupport` can support implementation without becoming public API.

## Library Products

```swift
.library(
    name: "DesignSystem",
    targets: ["DesignSystem"]
)
```

Use a library product for reusable modules imported by apps or other packages.

## Executable Products

```swift
.executable(
    name: "asset-tool",
    targets: ["AssetTool"]
)
```

Use executable products for tools and automation.

## Multi-Target Product

```swift
.library(
    name: "AppFeatures",
    targets: ["LoginFeature", "ProfileFeature"]
)
```

This exposes multiple modules together. Use carefully because product users now see more surface.

## Real iOS Use Case

```swift
let package = Package(
    name: "Checkout",
    products: [
        .library(name: "CheckoutFeature", targets: ["CheckoutFeature"]),
    ],
    targets: [
        .target(
            name: "CheckoutFeature",
            dependencies: ["CheckoutDomain", "PaymentClient"]
        ),
        .target(name: "CheckoutDomain"),
        .target(name: "PaymentClient"),
        .testTarget(
            name: "CheckoutFeatureTests",
            dependencies: ["CheckoutFeature"]
        ),
    ]
)
```

Here:

- `CheckoutFeature` is public product surface.
- `CheckoutDomain` and `PaymentClient` may be implementation modules.
- Tests depend on feature behavior.

## Senior iOS Engineer Artifact

```text
Artifact: Target/Product Boundary Map
Target:
Purpose:
Is it exposed as a product? yes/no
Who imports it?
Can it remain internal?
Public API risk:
Test target:
Dependency direction:
```

Senior lens:

- Not every target should be a product.
- Products are API commitments.
- Targets are architecture and build boundaries.
- Small targets can improve clarity but too many can add complexity.

## More Coding Examples

### Example 1: Internal Support Target

```swift
.target(name: "FeatureSupport"),
.target(
    name: "LoginFeature",
    dependencies: ["FeatureSupport"]
)
```

Only expose `LoginFeature` if clients do not need `FeatureSupport`.

### Example 2: Test Target With Dependencies

```swift
.testTarget(
    name: "LoginFeatureTests",
    dependencies: [
        "LoginFeature",
        "FeatureTestingSupport",
    ]
)
```

Testing support can be shared without shipping it as app API.

## Common Mistakes

- Exposing every target as a product.
- Mixing test utilities into production targets.
- Creating circular target dependencies.
- Naming products and targets inconsistently.
- Making one product contain unrelated feature modules.

## Interview Guide

Junior:

A target is a module. A product is what the package exposes to users.

Mid-level:

Use products for libraries or executables that clients need. Keep helper targets internal when possible.

Senior:

Target/product design controls module boundaries and API surface. I expose the smallest useful product and keep implementation targets hidden unless clients genuinely need them.

## Practice

1. Design targets for `AuthFeature`.
2. Decide which targets should be exposed as products.
3. Add a test support target.
4. Explain why product surface should stay small.
