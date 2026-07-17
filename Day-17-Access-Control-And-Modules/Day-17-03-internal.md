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

## Deep Dive Expansion

### Why This Topic Matters In Real Apps

internal is not only a syntax topic. In production Swift, it affects API surface, encapsulation, package boundaries, framework evolution, and source compatibility. A junior developer should be able to recognize the syntax and explain the basic behavior. A mid-level developer should know when the feature improves clarity or safety. A senior developer should understand how it changes API design, testability, performance, or long-term maintenance.

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

Use this checklist when applying internal in an app feature:

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

1. Write a minimal example that shows internal correctly.
2. Write a second example that demonstrates a common mistake.
3. Refactor the mistake into a safer version.
4. Explain the difference between the beginner solution and the production-ready solution.
5. Add one test case or reasoning check that proves the behavior.

## Applied Day-Level Example

The following example is not the only way to use internal, but it shows the kind of production shape you should connect this topic to:

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

## Senior iOS Engineer Artifact

### Senior Mental Model

For a senior iOS engineer, the topic **internal** is evaluated through this lens: access control is API economics; senior engineers decide what becomes contract and what stays replaceable. The goal is not to memorize the keyword or syntax in isolation. The goal is to understand how this topic changes correctness, ownership, testability, performance, and team-scale maintainability.

A senior engineer asks:

- What invariant does this protect?
- What future bug does this make harder to introduce?
- What does the call site communicate to another engineer?
- What is the runtime, memory, concurrency, or API-evolution cost?
- Can this design survive a larger feature, a second caller, or a module boundary?

### Artifact To Produce While Studying

Create this artifact for the topic:

```text
Artifact: an API surface artifact listing private, internal, package, public, and open declarations with justification
Topic: internal
Owner: Which type/module owns the decision?
Boundary: Is this local, feature-wide, package-wide, public API, actor-isolated, or UI-only?
Invariant: What must always remain true?
Failure Mode: What breaks if this is misused?
Verification: How would I prove it with tests, logs, compiler checks, or tooling?
```

This artifact forces senior-level thinking. It turns a language feature into an engineering decision with evidence.

### Senior-Level Scenario

Imagine designing feature modules, SDKs, design systems, Swift packages, and testing support. A junior answer usually explains what the syntax does. A senior answer explains the design pressure:

```text
I would first identify the owner and boundary. Then I would choose the smallest surface that preserves the invariant, keeps the call site readable, and avoids leaking implementation details. I would also check the failure path and decide how to verify the behavior with tests or tooling.
```

### What Can Go Wrong

The senior-level risk for this area is making helpers public for convenience or making classes open without supporting subclassing as a product feature.

That risk matters because iOS apps grow by accumulation. A small unclear decision can become a pattern copied into screens, services, tests, and public APIs.

### Review Checklist

Use this checklist in code review:

- Who needs this symbol?
- Is this public forever?
- Could package/internal solve the need?
- Is the example still understandable six months later?
- Does the implementation make the safe path easier than the unsafe path?
- If this appears in an interview, can I explain both the syntax and the tradeoff?

### Senior Interview Framing

Use this structure when answering at senior level:

```text
At the syntax level, this feature does X. At the design level, I use it to protect Y. The tradeoff is Z. In a real iOS app, I would apply it in this scenario, verify it this way, and avoid this common misuse.
```

For **internal**, fill it like this:

```text
At the syntax level, this topic gives me a Swift mechanism for a specific behavior. At the design level, I use it to make ownership, state, or boundaries clearer. The tradeoff is that misuse can hide complexity or create coupling. In a real iOS app, I would apply it where the invariant matters, verify it with focused tests or tooling, and avoid using it just because the syntax is available.
```

## More Coding Examples

These examples are intentionally small, but they are shaped like real app code. Use them to connect **internal** to code you might write in a SwiftUI/UIKit feature.

### Example 1: Public Entry, Private Implementation

```swift
public struct SettingsFeatureView: View {
    public init() { }

    public var body: some View {
        SettingsContentView()
    }
}

private struct SettingsContentView: View {
    var body: some View { Text("Settings") }
}
```

A module should expose the feature entry point, not every internal view.

### Example 2: Package-Level Helper

```swift
package struct TestUserFactory {
    package static func user() -> User {
        User(id: "1", name: "Test User")
    }
}
```

`package` is useful for shared test/support code inside a Swift package.

### How To Extend These Examples

- Add one failure path.
- Add one test case.
- Add one version that would be wrong in production and explain why.
- Explain what changes if this code moves from one screen into a shared module.

## Topic-Focused Mini Example

### Keep feature implementation inside a module

```swift
final class FeedMapper {
    func map(_ response: FeedResponse) -> FeedItem {
        FeedItem(id: response.id, title: response.title)
    }
}
```

Internal is the default and is often right for module implementation details.

### Why This Fits internal

This example is intentionally small so the core idea is easy to see. After understanding it, expand it with a failure path, a test case, and one realistic constraint from a production iOS feature.

