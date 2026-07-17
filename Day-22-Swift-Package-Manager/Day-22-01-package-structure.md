# Day 22: Package Structure

## What Swift Package Manager Is

Swift Package Manager, usually called SwiftPM or SPM, is Swift's official tool for:

- Defining Swift packages
- Building libraries and executables
- Managing dependencies
- Running tests
- Bundling resources
- Running plugins

A package is described by a `Package.swift` manifest.

```swift
// swift-tools-version: 6.2

import PackageDescription

let package = Package(
    name: "NetworkingKit",
    products: [
        .library(name: "NetworkingKit", targets: ["NetworkingKit"]),
    ],
    targets: [
        .target(name: "NetworkingKit"),
        .testTarget(name: "NetworkingKitTests", dependencies: ["NetworkingKit"]),
    ]
)
```

## Standard Package Layout

Typical layout:

```text
NetworkingKit/
  Package.swift
  Sources/
    NetworkingKit/
      APIClient.swift
      Endpoint.swift
  Tests/
    NetworkingKitTests/
      APIClientTests.swift
  README.md
```

Important directories:

- `Sources`: production source code
- `Tests`: test source code
- `Package.swift`: manifest
- `Resources`: optional resource files inside a target folder

## Why Structure Matters

SPM package structure affects:

- Build boundaries
- Module names
- Public API exposure
- Testability
- Dependency direction
- Reuse across apps
- CI and release workflows

A senior iOS engineer does not treat package structure as file organization only. It is architecture.

## Minimal Library Package

```swift
// swift-tools-version: 6.2
import PackageDescription

let package = Package(
    name: "DesignSystem",
    platforms: [
        .iOS(.v17),
    ],
    products: [
        .library(name: "DesignSystem", targets: ["DesignSystem"]),
    ],
    targets: [
        .target(name: "DesignSystem"),
        .testTarget(name: "DesignSystemTests", dependencies: ["DesignSystem"]),
    ]
)
```

Use this when you want app modules to import reusable UI components.

## Minimal Executable Package

```swift
let package = Package(
    name: "ReleaseTool",
    platforms: [.macOS(.v14)],
    products: [
        .executable(name: "release-tool", targets: ["ReleaseTool"]),
    ],
    targets: [
        .executableTarget(name: "ReleaseTool"),
    ]
)
```

Useful for command-line tooling, automation, and CI helpers.

## Real iOS Use Case

An iOS workspace might split code like this:

```text
App target
  imports LoginFeature
  imports DesignSystem
  imports NetworkingKit

Packages/
  LoginFeature
  DesignSystem
  NetworkingKit
  SharedModels
```

This helps teams work independently and limits accidental dependencies.

## Senior iOS Engineer Artifact

```text
Artifact: Package Structure Review
Package name:
Purpose: app feature / library / tool / test support
Products:
Targets:
Public modules:
Internal targets:
Test targets:
Dependency direction:
Platform minimums:
Release/versioning strategy:
```

Senior lens:

- Packages should express architecture boundaries.
- Avoid one giant shared package that becomes a dumping ground.
- Keep public products small and intentional.
- Use separate targets for implementation, testing support, plugins, and executable tools when needed.

## More Coding Examples

### Example 1: Feature Package

```swift
let package = Package(
    name: "ProfileFeature",
    platforms: [.iOS(.v17)],
    products: [
        .library(name: "ProfileFeature", targets: ["ProfileFeature"]),
    ],
    targets: [
        .target(
            name: "ProfileFeature",
            dependencies: ["DesignSystem", "NetworkingKit"]
        ),
        .testTarget(
            name: "ProfileFeatureTests",
            dependencies: ["ProfileFeature"]
        ),
    ]
)
```

This package exposes only the feature module.

### Example 2: Shared Test Support

```text
Sources/
  FeatureTestingSupport/
    StubUserService.swift
Tests/
  LoginFeatureTests/
```

Shared test support should usually be its own target, not mixed into production code.

## Common Mistakes

- Putting every file into one target.
- Exposing helper targets as products unnecessarily.
- Making tests depend on app targets that are hard to build.
- Forgetting platform minimums.
- Creating circular dependencies between feature modules.
- Treating package layout as folder cleanup instead of architecture.

## Interview Guide

Junior:

A Swift package has a `Package.swift` file plus `Sources` and `Tests` folders.

Mid-level:

Targets become modules. Products expose targets to package users. Tests usually live in test targets.

Senior:

Package structure is a modular architecture decision. I use it to enforce dependency direction, isolate features, reduce API surface, and improve build/test boundaries.

## Practice

1. Create a library package structure for `AnalyticsKit`.
2. Add a test target.
3. Decide which targets should be products.
4. Draw dependency direction between `App`, `DesignSystem`, and `NetworkingKit`.
