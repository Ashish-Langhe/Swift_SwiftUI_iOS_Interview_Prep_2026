# Day 17: Module Boundaries

## What A Module Boundary Means

A module boundary separates one compiled unit of Swift code from another.

Common modules:

- App target
- Framework target
- Swift package target
- Test target
- Extension target
- Widget target

When code crosses a module boundary, access control matters.

```swift
import Networking
import DesignSystem
import LoginFeature
```

Each imported target is a module.

## Beginner Mental Model

A module is like a room.

- `private`: visible inside one small area
- `fileprivate`: visible inside one file
- `internal`: visible inside the current room
- `package`: visible to rooms inside the same package
- `public`: visible to other rooms
- `open`: visible and subclassable from other rooms

## Why Module Boundaries Matter

Module boundaries help you:

- Reduce coupling
- Speed up understanding
- Protect implementation details
- Reuse code across apps
- Build feature modules
- Test code in isolation
- Control public API

Without boundaries, every type can easily depend on every other type.

## Single-Target App

```text
MyApp target
  LoginView.swift
  ProfileView.swift
  APIClient.swift
  UserSession.swift
```

In a single app target:

- Most declarations are internal by default.
- Every file can see internal declarations.
- Access control still matters for type-level encapsulation.

Use `private` aggressively even in a single-target app.

## Multi-Module App

```text
App
  imports LoginFeature
  imports ProfileFeature
  imports DesignSystem
  imports Networking

LoginFeature
  imports DesignSystem
  imports Networking

ProfileFeature
  imports DesignSystem
  imports Networking
```

Now access control becomes architectural.

`LoginFeature` should expose only what the app shell needs.

```swift
public struct LoginScreen: View {
    public init() { }

    public var body: some View {
        LoginContentView()
    }
}

private struct LoginContentView: View {
    var body: some View {
        Text("Login")
    }
}
```

The app imports `LoginScreen`, not every login implementation detail.

## Module Boundary Example

Networking module:

```swift
public protocol APIClient {
    func send<Request: APIRequest>(_ request: Request) async throws -> Request.Response
}

internal final class URLSessionAPIClient: APIClient {
    func send<Request: APIRequest>(_ request: Request) async throws -> Request.Response {
        fatalError("Implementation")
    }
}
```

External modules depend on `APIClient`.

The concrete implementation can stay internal.

## Feature Boundary Example

```swift
public struct CheckoutFeature: View {
    private let dependencies: CheckoutDependencies

    public init(dependencies: CheckoutDependencies) {
        self.dependencies = dependencies
    }

    public var body: some View {
        CheckoutRootView(dependencies: dependencies)
    }
}

public protocol PaymentServicing {
    func startPayment() async throws
}

public struct CheckoutDependencies {
    public let paymentService: PaymentServicing

    public init(paymentService: PaymentServicing) {
        self.paymentService = paymentService
    }
}
```

Only the feature entry point and dependency contract are public.

Internal screens, mappers, reducers, and helpers remain hidden.

## Access Levels Across Boundaries

```text
Same type/scope: private
Same file: fileprivate
Same target/module: internal
Same package: package
Other module: public
Other module subclassing: open
```

If another module cannot see a type, it cannot use it in public API.

## `@testable import` Boundary

Tests usually import the app or package module:

```swift
@testable import LoginFeature
```

This allows access to internal declarations.

But tests still should not overfit to every implementation detail. Too many tests of internal details can make refactoring painful.

Senior approach:

- Test public behavior through public APIs where possible.
- Use `@testable` for important internal units.
- Keep private helpers tested through the behavior that uses them.

## Extension Targets

iOS apps often have extension modules:

- Widget extension
- Share extension
- Notification service extension
- App Intent extension

These are separate targets.

If an extension needs shared code, move that code into a shared framework/package target.

```text
App target
Widget target
SharedModels target
```

`internal` code in the app target is not visible to the widget target.

## Build And API Design Impact

Module boundaries affect:

- What needs recompilation
- What can be reused
- What can be tested separately
- What APIs need source stability
- How much code a feature can accidentally reach

Good boundaries make teams faster because code has fewer hidden dependencies.

## Advanced Design Notes

Senior engineers use modules to create architectural pressure.

Example:

```text
Feature modules cannot import App.
Networking cannot import UI.
DesignSystem cannot import feature modules.
Domain should not depend on UIKit.
```

Access control supports these rules.

But module boundaries also have costs:

- More package/target setup
- More explicit dependencies
- More public or package API design
- Possible build configuration complexity

Use modules where they create real clarity.

## Modern Swift 6.x Notes

Modern Swift development uses module boundaries heavily through Swift Package Manager.

Relevant modern points:

- `package` access helps share declarations across package targets.
- Concurrency annotations in public APIs cross module boundaries.
- `Sendable` conformance may become visible in public API design.
- `@MainActor` on public UI APIs affects callers.
- Smaller modules can make strict concurrency migration easier because shared state is more localized.

## Junior Interview Answer

A module is a compiled unit of code, such as an app target, framework, package target, or test target. Access control decides what can be used inside or outside that module.

## Mid-Level Interview Answer

Module boundaries let us hide implementation details and expose only the APIs other modules need. `internal` stays inside one module, `package` works across package targets, and `public` crosses module boundaries.

## Senior Interview Answer

Module boundaries are architecture boundaries. They control dependency direction, API surface, build isolation, and testability. I expose small public entry points, keep implementation internal or package-level, and avoid leaking DTOs or infrastructure details across features.

## Points To Remember

- App targets, frameworks, packages, tests, widgets, and extensions are modules.
- `internal` does not cross module boundaries.
- `package` crosses targets in the same package.
- `public` crosses module boundaries.
- Module boundaries should reflect architecture, not random file organization.

## Practice

1. Draw modules for an app with Login, Profile, Networking, and DesignSystem.
2. Decide which declarations should be public, package, internal, or private.
3. Explain why a widget cannot access internal app target code.
4. Design a feature module with one public entry screen and internal implementation.
