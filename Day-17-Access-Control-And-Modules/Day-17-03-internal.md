# Day 17: `internal`

## What `internal` Means

`internal` is Swift's default access level.

An internal declaration can be used anywhere inside the same module, but not from another module.

```swift
struct UserDTO {
    let id: String
    let name: String
}
```

This is the same as:

```swift
internal struct UserDTO {
    internal let id: String
    internal let name: String
}
```

## What Is A Module?

A module is a compiled unit of Swift code.

Examples:

- An iOS app target
- A framework target
- A Swift package target
- A test target
- A command-line executable target

If your app has one target, most files are in the same module.

If your code is split into `Networking`, `DesignSystem`, `FeatureLogin`, and `App`, each target is usually a separate module.

## Beginner Mental Model

Use `internal` when you want:

```text
Everyone inside this module can use it, but outside modules cannot.
```

Because it is the default, you often do not write it explicitly.

## Real iOS Use Cases

Internal is common for:

- App-only view models
- Feature-specific models
- Use cases inside a target
- DTOs used by a networking module
- Helper services not exposed outside a framework

```swift
struct FeedItemResponse: Decodable {
    let id: String
    let title: String
}

final class FeedMapper {
    func map(_ response: FeedItemResponse) -> FeedItem {
        FeedItem(id: response.id, title: response.title)
    }
}
```

If this code lives inside `FeedFeature`, external modules do not need to know about `FeedItemResponse` or `FeedMapper`.

## Internal And App Targets

In a normal single-target iOS app, `internal` often feels like "available everywhere."

```swift
final class AppRouter {
    func showHome() { }
}
```

Every file in the app target can use `AppRouter`.

That is convenient, but it can also make code too reachable. Use `private` or `fileprivate` for implementation details.

## Internal And Test Targets

Tests are usually a separate module. That means tests cannot access internal declarations by default.

Swift supports `@testable import`:

```swift
@testable import MyApp
```

With testable import, tests can access internal declarations from the imported module.

```swift
final class PriceCalculatorTests: XCTestCase {
    func testDiscount() {
        let calculator = PriceCalculator()
        XCTAssertEqual(calculator.discountedPrice(100), 90)
    }
}
```

Important:

- `@testable` exposes internal declarations to tests.
- It does not expose private declarations.
- It is meant for testing, not production module design.

## Internal Protocols

Internal protocols are useful inside a module.

```swift
protocol TokenRefreshing {
    func refreshToken() async throws -> String
}

final class AuthService: TokenRefreshing {
    func refreshToken() async throws -> String {
        "new-token"
    }
}
```

If no outside module needs the protocol, keep it internal.

## Access Level Inference

A declaration cannot be more visible than the types it uses.

```swift
internal struct InternalModel { }

public func makeModel() -> InternalModel {
    InternalModel()
}
```

This is invalid because a public function cannot expose an internal return type.

Better:

```swift
public struct PublicModel { }

public func makeModel() -> PublicModel {
    PublicModel()
}
```

## Advanced Design Notes

Internal is great for implementation inside module boundaries, but large app targets can make `internal` too broad.

If a target has hundreds of files, an internal type may become an accidental shared dependency.

Senior-level design questions:

- Should this type be visible to the whole module?
- Is this really a feature-local implementation detail?
- Should this code live in a smaller module?
- Should this be private but exposed through a narrow protocol?

## Internal In Multi-Module Architecture

```text
App module
  imports LoginFeature
  imports Networking

LoginFeature module
  internal LoginViewModel
  public LoginView

Networking module
  public APIClient
  internal URLSessionTransport
```

`internal` lets each module hide implementation while exposing only what other modules need.

## Modern Swift 6.x Notes

`internal` remains the default access level.

Modern Swift makes module boundaries more important because:

- Swift packages often contain multiple targets.
- `package` access gives another layer between `internal` and `public`.
- Strict concurrency pushes teams to isolate shared mutable state carefully.
- Public APIs need more care because concurrency annotations can become part of the API contract.

## Junior Interview Answer

`internal` means accessible anywhere in the same module. It is Swift's default access level.

## Mid-Level Interview Answer

I use internal for declarations that are shared across files inside a module but should not be exposed outside that module. Tests can access internal declarations using `@testable import`.

## Senior Interview Answer

`internal` is the default module-wide visibility, but default does not always mean best. In a large app target, internal APIs can create broad accidental coupling. In modular architecture, internal is valuable because it keeps implementation details inside a feature, package target, or framework while exposing only carefully designed public APIs.

## Points To Remember

- `internal` is the default.
- It is visible only inside the same module.
- `@testable import` exposes internal declarations to tests.
- Public APIs cannot expose internal types.
- In large targets, internal can still be too broad.

## Practice

1. Explain why tests need `@testable import` for internal members.
2. Create an internal DTO and a public domain model.
3. Refactor an internal helper into private if only one type uses it.
4. Draw the module boundary for an app, a feature module, and a networking module.
