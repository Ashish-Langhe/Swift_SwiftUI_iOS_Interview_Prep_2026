# Day 22: Dependencies

## What Dependencies Are

Dependencies are packages or targets your target needs to build.

External package dependency:

```swift
dependencies: [
    .package(url: "https://github.com/apple/swift-collections.git", from: "1.1.0"),
]
```

Target dependency:

```swift
.target(
    name: "Feature",
    dependencies: [
        .product(name: "Collections", package: "swift-collections"),
        "FeatureDomain",
    ]
)
```

## Version Requirements

Common version styles:

```swift
.package(url: "...", from: "1.2.0")
.package(url: "...", exact: "1.2.3")
.package(url: "...", branch: "main")
.package(url: "...", revision: "abc123")
```

For production apps, prefer semantic version ranges when possible.

Use exact/branch/revision carefully because they can make updates or reproducibility harder.

## Product Dependencies

An external package can expose multiple products.

```swift
.product(name: "Collections", package: "swift-collections")
```

The product name is what your target imports.

## Dependency Direction

Healthy direction:

```text
Feature -> Domain -> SharedModels
Feature -> Networking
Feature -> DesignSystem
```

Risky direction:

```text
Networking -> Feature
DesignSystem -> App
Domain -> UIKit-heavy feature
```

Senior engineers use packages to enforce dependency direction.

## Real iOS Use Case

```swift
.target(
    name: "ProfileFeature",
    dependencies: [
        "ProfileDomain",
        "DesignSystem",
        .product(name: "Collections", package: "swift-collections"),
    ]
)
```

This makes the feature's dependency graph explicit.

## Dependency Pinning

SPM resolves dependencies into `Package.resolved`.

Commit `Package.resolved` for app repositories so CI and teammates build the same dependency versions.

For libraries, teams may choose differently depending on release strategy.

## Senior iOS Engineer Artifact

```text
Artifact: Dependency Review
Dependency:
Version rule:
Used by target:
Reason:
Transitive dependency risk:
License/security review:
Update policy:
Can this be replaced by standard library/Foundation?
```

Senior lens:

- Every dependency is a maintenance decision.
- Check license, health, binary size, API stability, and transitive dependencies.
- Avoid adding dependencies for tiny utilities.
- Keep dependency direction clean.

## More Coding Examples

### Example 1: Add A Package

```swift
dependencies: [
    .package(url: "https://github.com/apple/swift-collections.git", from: "1.1.0"),
]
```

### Example 2: Use A Product

```swift
.target(
    name: "CacheKit",
    dependencies: [
        .product(name: "Collections", package: "swift-collections"),
    ]
)
```

## Common Mistakes

- Depending on a branch in production.
- Adding a package without checking license or maintenance.
- Letting feature modules depend on each other randomly.
- Forgetting `Package.resolved` in app repos.
- Importing a package product into targets that do not need it.

## Interview Guide

Junior:

Dependencies let your package use code from other packages or targets.

Mid-level:

Add package dependencies in the manifest and attach package products to specific targets.

Senior:

Dependency management is risk management. I consider versioning, security, license, transitive dependencies, build impact, and architectural direction before adding a package.

## Practice

1. Add `swift-collections` to a package.
2. Attach one product to one target only.
3. Explain `from` vs `exact`.
4. Review whether a utility dependency is worth adding.
