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

## Deep Dive Expansion

### Why This Topic Matters In Real Apps

Module Boundaries is not only a syntax topic. In production Swift, it affects API surface, encapsulation, package boundaries, framework evolution, and source compatibility. A junior developer should be able to recognize the syntax and explain the basic behavior. A mid-level developer should know when the feature improves clarity or safety. A senior developer should understand how it changes API design, testability, performance, or long-term maintenance.

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

Use this checklist when applying Module Boundaries in an app feature:

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

1. Write a minimal example that shows Module Boundaries correctly.
2. Write a second example that demonstrates a common mistake.
3. Refactor the mistake into a safer version.
4. Explain the difference between the beginner solution and the production-ready solution.
5. Add one test case or reasoning check that proves the behavior.

## Applied Day-Level Example

The following example is not the only way to use Module Boundaries, but it shows the kind of production shape you should connect this topic to:

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

For a senior iOS engineer, the topic **Module Boundaries** is evaluated through this lens: access control is API economics; senior engineers decide what becomes contract and what stays replaceable. The goal is not to memorize the keyword or syntax in isolation. The goal is to understand how this topic changes correctness, ownership, testability, performance, and team-scale maintainability.

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
Topic: Module Boundaries
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

For **Module Boundaries**, fill it like this:

```text
At the syntax level, this topic gives me a Swift mechanism for a specific behavior. At the design level, I use it to make ownership, state, or boundaries clearer. The tradeoff is that misuse can hide complexity or create coupling. In a real iOS app, I would apply it where the invariant matters, verify it with focused tests or tooling, and avoid using it just because the syntax is available.
```

