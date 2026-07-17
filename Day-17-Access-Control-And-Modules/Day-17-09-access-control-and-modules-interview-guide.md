# Day 17: Access Control And Modules Interview Guide

## One-Minute Interview Answer

Swift access control decides where declarations can be used. `private` keeps implementation inside a scope, `fileprivate` keeps it inside a file, `internal` keeps it inside a module, `package` shares it across targets in the same Swift package, `public` exposes it to other modules, and `open` additionally allows external subclassing and overriding. In real iOS architecture, access control is not just syntax. It controls module boundaries, framework API design, testability, and long-term source compatibility.

## Access Level Order

```text
Most restrictive

private
fileprivate
internal
package
public
open

Least restrictive
```

Important:

- `open` applies to classes and class members.
- `internal` is the default.
- `package` is for Swift package boundaries.
- `public` does not mean subclassable.

## Full Learning Path: Basic To Advanced

### Level 1: Basic Definitions

Know what each level means.

```text
private: same declaration scope
fileprivate: same file
internal: same module
package: same package
public: other modules can use
open: other modules can subclass/override
```

### Level 2: Practical iOS Usage

Know where each one appears.

```swift
@State private var isLoading = false
private(set) var items: [Item] = []
internal final class FeedMapper { }
public struct PrimaryButton: View { }
open class BaseViewController: UIViewController { }
```

### Level 3: Module Architecture

Understand:

- App targets
- Framework targets
- Swift package targets
- Test targets
- Extension targets
- Public API boundaries

### Level 4: Senior Framework Design

Be able to explain:

- Public API is a contract.
- Open API is an inheritance contract.
- Public protocols are harder to evolve.
- `package` avoids public API pollution.
- Access control protects invariants and reduces coupling.

## Quick Comparison Table

| Access level | Visible where | Best use |
|---|---|---|
| `private` | Same scope and same-file extensions of the declaration | Local state, helpers, invariants |
| `fileprivate` | Same source file | File-local helper types |
| `internal` | Same module | App/feature implementation |
| `package` | Same Swift package | Shared package infrastructure |
| `public` | Other modules | Supported external API |
| `open` | Other modules with subclassing/overriding | Inheritance-based framework extension |

## `private`

Interview answer:

`private` hides implementation details inside the current declaration scope. I use it for state and helpers that should not be touched externally.

Example:

```swift
final class SessionStore {
    private(set) var token: String?

    func update(token: String) {
        self.token = token
    }
}
```

Senior point:

`private` protects invariants. External code can read `token`, but only `SessionStore` controls mutation.

## `fileprivate`

Interview answer:

`fileprivate` makes a declaration visible across the same file. I use it rarely, mostly for helper types or extensions that belong only to one source file.

Example:

```swift
fileprivate struct PriceFormatter {
    func string(from value: Decimal) -> String {
        "\(value)"
    }
}
```

Senior point:

Overusing `fileprivate` can hide coupling. If unrelated types need file-private access to each other, the design may need a cleaner interface.

## `internal`

Interview answer:

`internal` is Swift's default. It allows access anywhere inside the same module.

Example:

```swift
final class FeedMapper {
    func map(_ response: FeedResponse) -> FeedItem {
        FeedItem(id: response.id, title: response.title)
    }
}
```

Senior point:

In a large app target, internal can still be broad. In modular architecture, internal is powerful because it hides implementation inside each feature or framework target.

## `package`

Interview answer:

`package` allows access across targets in the same Swift package, but hides the declaration from external package consumers.

Example:

```swift
package enum DesignTokens {
    package static let cornerRadius: CGFloat = 8
}
```

Senior point:

`package` is ideal for multi-target package internals. It prevents the common mistake of making support APIs public just because another package target needs them.

## `public`

Interview answer:

`public` exposes a declaration to other modules.

Example:

```swift
public struct User {
    public let id: String

    public init(id: String) {
        self.id = id
    }
}
```

Senior point:

A public type's members and initializers are not automatically public. Public API is a source compatibility promise, so keep it small and intentional.

## `open`

Interview answer:

`open` allows external modules to subclass a class and override open members.

Example:

```swift
open class BaseScreen: UIViewController {
    open func configureLayout() { }
}
```

Senior point:

Open is a stronger promise than public. It should be used only when inheritance is an intentional customization mechanism. Otherwise prefer `public final`, protocols, or composition.

## Most Common Interview Traps

### Trap 1: Public Type, Internal Init

```swift
public struct Product {
    public let id: String
}
```

External module may not be able to initialize this type using a memberwise initializer.

Fix:

```swift
public struct Product {
    public let id: String

    public init(id: String) {
        self.id = id
    }
}
```

### Trap 2: Public Does Not Mean Open

```swift
public class SDKScreen: UIViewController { }
```

External modules can use it but cannot subclass it.

Use `open` only if subclassing is supported.

### Trap 3: Access Level Cannot Expose Hidden Types

```swift
internal struct InternalModel { }

public func makeModel() -> InternalModel {
    InternalModel()
}
```

A public function cannot expose an internal return type.

### Trap 4: Access Control Is Not Security

`private` prevents source-level access. It does not encrypt data or protect secrets at runtime.

Do not store API secrets in code just because a property is private.

### Trap 5: Access Control Is Not Thread Safety

```swift
private var count = 0
```

This hides `count`, but it does not make concurrent access safe.

Use actors, locks, serial queues, or proper isolation where needed.

## Real iOS Architecture Scenario

Question:

You are building a `LoginFeature` module. What should be public?

Strong answer:

```text
Public:
  LoginView
  LoginConfiguration
  LoginResult

Internal:
  LoginViewModel
  LoginMapper
  LoginValidator

Private:
  SwiftUI helper views used by one file

Package:
  Shared test factories or support code used by sibling package targets
```

Explanation:

The app shell needs the entry point and configuration. It does not need validators, mappers, DTOs, or internal view hierarchy.

## Junior-Level Questions

What is the default access level in Swift?

`internal`.

What does `private` mean?

It restricts access to the current declaration scope and same-file extensions of that declaration.

What does `public` mean?

Other modules can use the declaration.

## Mid-Level Questions

What is the difference between `private` and `fileprivate`?

`private` is scoped to the declaration. `fileprivate` is visible anywhere in the same file.

What is the difference between `internal` and `package`?

`internal` is visible inside one module or target. `package` is visible across targets in the same Swift package.

What is the difference between `public` and `open`?

`public` allows external use. `open` allows external subclassing and overriding.

## Senior-Level Questions

How do access control decisions affect framework evolution?

Every public or open declaration becomes an API contract. Removing it, renaming it, changing labels, changing generic constraints, or adding protocol requirements can break external users. That is why implementation details should stay private, internal, or package.

When would you use `package`?

When multiple targets inside the same Swift package need shared implementation, but external package consumers should not see that API.

When would you avoid `open`?

I avoid `open` unless inheritance is explicitly supported. Open classes are harder to evolve because external subclasses may depend on behavior. I usually prefer `public final`, protocols, injected dependencies, or closures.

## Senior System Design Answer

For a modular iOS app, I treat access control as architecture enforcement. Feature modules expose public entry points and dependency contracts. Implementation details stay internal. Shared support across package targets uses package access. Local state and helpers stay private. Framework APIs stay small and stable. I use open only for deliberate subclassing hooks, and I document override behavior. With Swift concurrency, I also consider whether public APIs should be `@MainActor`, `Sendable`, async, or actor-based, because those choices become part of the external contract.

## Latest Swift Notes

As of the verified Swift 6.x material used for this curriculum:

- Swift has six access levels: `private`, `fileprivate`, `internal`, `package`, `public`, and `open`.
- `package` is the major modern access-control level to know for SwiftPM architecture.
- Swift 6.x concurrency makes public API design more important because isolation and sendability can affect callers.
- There is no need to mark everything public in a package; prefer `package` for shared internals.

## Points To Remember

- Default access is `internal`.
- Start with the most restrictive access that works.
- Public API should be small and stable.
- Open API should be rare and intentional.
- `package` is useful for multi-target packages.
- `private(set)` is common for controlled mutation.
- Access control is compile-time visibility, not runtime security.
- Access control is not concurrency safety.

## Practice Prompts

1. Design access levels for a `DesignSystem` framework.
2. Explain why public structs need explicit public initializers.
3. Convert a public helper type into package access.
4. Explain why `open` is harder to maintain than `public final`.
5. Design a feature module with one public screen and internal implementation.
6. Explain how `@testable import` changes access in tests.
