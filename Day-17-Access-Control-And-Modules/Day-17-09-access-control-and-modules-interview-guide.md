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

## Deep Dive Expansion

### Why This Topic Matters In Real Apps

Access Control And Modules Interview Guide is not only a syntax topic. In production Swift, it affects API surface, encapsulation, package boundaries, framework evolution, and source compatibility. A junior developer should be able to recognize the syntax and explain the basic behavior. A mid-level developer should know when the feature improves clarity or safety. A senior developer should understand how it changes API design, testability, performance, or long-term maintenance.

In iOS work, this usually appears while feature modules, design systems, SDK entry points, and internal helpers. The important question is not "Can I write the syntax?" The important question is "Does this code make invalid states harder, make ownership clearer, and make future changes safer?"

### Beginner To Senior Progression

Beginner level:

- Define the concept in plain language.
- Write the smallest working example.
- Recognize the compiler error when the feature is used incorrectly.
- Explain what happens at the call site.

Mid-level level:

- Choose this feature over a nearby alternative for a clear reason.
- Handle edge cases, nil/error/cancellation/performance concerns where relevant.
- Keep the code readable for the next developer.
- Write tests around the behavior, not just the implementation detail.

Senior level:

- Explain the design tradeoff and the failure mode it prevents.
- Understand how this feature behaves across module, actor, memory, or API boundaries.
- Design examples that scale from a small screen to a larger feature.
- Avoid exposing implementation details as permanent API.

### Production-Style Example Pattern

Use this checklist when applying Access Control And Modules Interview Guide in an app feature:

1. Identify the owner of the data or behavior.
2. Decide whether the value should be mutable, immutable, optional, throwing, async, isolated, or private.
3. Keep the public surface small and intention-revealing.
4. Add one realistic failure path, not only the happy path.
5. Check whether the code is still understandable from the call site.

```swift
struct FeatureState: Equatable {
    var isLoading: Bool
    var message: String?
    var canRetry: Bool
}

func makeInitialState() -> FeatureState {
    FeatureState(isLoading: false, message: nil, canRetry: false)
}
```

This small pattern is intentionally simple: define the state, control mutation through a narrow function, and make the result easy to inspect in tests.

### Edge Cases To Think About

- What happens when the input is empty, nil, duplicated, delayed, or invalid?
- What happens when this code is called repeatedly from a scrolling list or fast-changing UI?
- Does this API expose too much mutable state?
- Does the implementation assume a specific ordering, lifetime, actor, or thread?
- Will this still be easy to test after the feature grows?

### Topic-Specific Senior Notes

- Turn each answer into a story: situation, decision, tradeoff, result, and verification.
- For senior interviews, avoid only definitions. Explain why the design prevents bugs in a real app.
- Start restrictive and widen access only when another boundary genuinely needs it.
- Treat public and open declarations as compatibility promises, not convenience keywords.

### Common Interview Follow-Ups

Be ready for these follow-ups:

- Why did you choose this approach instead of the simpler alternative?
- What bug could happen if this is implemented carelessly?
- How would this behave in a large codebase with multiple modules?
- How would you test this without relying on UI screenshots?
- What changes when this code becomes async, public, generic, or shared?

### Strong Interview Framing

A strong answer should sound like this:

```text
I understand the basic syntax, but I also think about ownership and boundaries. I choose the approach that keeps state valid, makes the call site clear, and avoids unnecessary coupling. In a production iOS app, I would also consider testing, cancellation or error behavior where relevant, and whether this should remain an implementation detail or become part of a public API.
```

### Extra Practice

1. Write a minimal example that shows Access Control And Modules Interview Guide correctly.
2. Write a second example that demonstrates a common mistake.
3. Refactor the mistake into a safer version.
4. Explain the difference between the beginner solution and the production-ready solution.
5. Turn one answer into a real project story with tradeoffs and verification.

## Applied Day-Level Example

The following example is not the only way to use Access Control And Modules Interview Guide, but it shows the kind of production shape you should connect this topic to:

```swift
public struct LoginFeatureView: View {
    private let configuration: LoginConfiguration

    public init(configuration: LoginConfiguration) {
        self.configuration = configuration
    }

    public var body: some View {
        LoginContentView(configuration: configuration)
    }
}

private struct LoginContentView: View {
    let configuration: LoginConfiguration
    var body: some View { Text(configuration.title) }
}
```

Access control keeps framework API small. The app needs the feature entry point, not every helper view inside the feature.

### How To Study This Example

- Identify which part is syntax and which part is design.
- Ask what invalid state the example prevents.
- Ask what would need to change if the feature became async, generic, public, or shared across modules.
- Rewrite the example with one deliberate bug, then explain how a reviewer should catch it.
- Turn the example into a two-minute interview explanation with tradeoffs.

