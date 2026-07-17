# Day 22: Package Traits From Swift 6.1

## What Package Traits Are

Swift 6.1 introduced package traits.

Traits let a package offer configurable API/features for different environments or client needs.

Use cases:

- Optional features
- Experimental APIs
- Embedded Swift support
- WebAssembly support
- Optional dependencies
- Platform-specific capability groups

## Basic Trait Declaration

```swift
let package = Package(
    name: "ImageKit",
    traits: [
        .trait(name: "RemoteLoading"),
        .default(enabledTraits: ["RemoteLoading"]),
    ],
    targets: [
        .target(name: "ImageKit"),
    ]
)
```

If `RemoteLoading` is enabled, the package can conditionally compile APIs.

## Conditional Compilation

Inside the package:

```swift
#if RemoteLoading
public struct RemoteImageLoader {
    public init() { }
}
#endif
```

Traits expose conditional compilation flags within the package that defines them.

## Using Traits In Dependencies

A client can choose traits when declaring the dependency:

```swift
.package(
    url: "https://github.com/example/ImageKit.git",
    from: "1.0.0",
    traits: [
        .default,
        "ExperimentalFilters",
    ]
)
```

`.default` enables the package's default traits.

## Important Rules

From the official SwiftPM trait model:

- Traits are namespaced within the package.
- Trait names must be valid identifiers, with `-` and `+` also allowed.
- Do not use `default` or `defaults` as trait names.
- Traits activated while building a package activate only within that package.
- If your package needs a dependency trait, declare it in the dependency.
- Do not remove or disable public API when enabling a trait.

## Real iOS Use Case

```swift
traits: [
    .trait(name: "DebugTools"),
    .trait(name: "PreviewContent"),
    .default(enabledTraits: ["PreviewContent"]),
]
```

Feature code:

```swift
#if DebugTools
public struct DebugFeatureOverlay { }
#endif
```

This lets debug tools stay configurable without becoming required for every client.

## Optional Dependencies

Traits can express optional functionality that requires additional dependencies.

Senior caution:

If enabling a trait adds a dependency, document the impact clearly.

## Senior iOS Engineer Artifact

```text
Artifact: Package Trait Design
Trait name:
Default enabled? yes/no
Feature enabled:
Additional dependency:
Public API added:
Compilation condition:
Client configuration example:
Compatibility risk:
```

Senior lens:

- Traits are API-surface configuration.
- They should add capability, not remove expected public API.
- Defaults should be safe for most clients.
- Trait names need long-term clarity.

## More Coding Examples

### Example 1: Experimental API

```swift
traits: [
    .trait(name: "ExperimentalSearch"),
]
```

```swift
#if ExperimentalSearch
public struct ExperimentalSearchEngine { }
#endif
```

### Example 2: Grouped Trait

Traits can represent feature groups. Use grouping when clients often enable features together.

```swift
traits: [
    .trait(name: "ImageLoading"),
    .trait(name: "ImageCaching"),
    .trait(name: "FullImagePipeline", enabledTraits: ["ImageLoading", "ImageCaching"]),
]
```

## Common Mistakes

- Using traits to remove public API.
- Making experimental traits default without clear commitment.
- Forgetting to document client configuration.
- Assuming a trait in one package affects another package.
- Creating vague names like `Feature1`.

## Interview Guide

Junior:

Package traits let packages offer configurable features.

Mid-level:

Declare traits in `Package.swift`, use conditional compilation inside the package, and let clients opt into traits in dependency declarations.

Senior:

Traits are configurable API design. I use them for optional capabilities while preserving compatibility, documenting defaults, and avoiding surprising dependency changes.

## Practice

1. Add a `DebugTools` trait.
2. Use `#if DebugTools`.
3. Configure a dependency with `.default` plus one trait.
4. Explain why enabling a trait should not remove public API.
