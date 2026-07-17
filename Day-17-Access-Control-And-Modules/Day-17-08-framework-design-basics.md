# Day 17: Framework Design Basics

## What Framework Design Means

Framework design is the practice of deciding what another module can use, customize, and depend on.

In Swift, this is strongly connected to access control.

Framework API design answers:

- What should be public?
- What should be open?
- What should stay internal or package?
- What should be private implementation?
- What can change later without breaking users?

## Beginner Mental Model

A framework should feel like a clean toolkit.

Consumers should see:

- Clear entry points
- Stable types
- Useful configuration
- Helpful protocols
- Minimal implementation details

They should not see:

- Random helpers
- Backend DTOs
- Temporary mappers
- Internal caches
- Experimental types

## Basic Framework Shape

```text
DesignSystem framework
  Public:
    PrimaryButton
    AppTextField
    AppTheme

  Internal:
    ButtonStyleResolver
    ThemeStorage
    ColorParser

  Private:
    One-type helper methods
```

Public API should be intentionally small.

## Public Entry Points

```swift
public struct PrimaryButton: View {
    private let title: String
    private let action: () -> Void

    public init(_ title: String, action: @escaping () -> Void) {
        self.title = title
        self.action = action
    }

    public var body: some View {
        Button(title, action: action)
            .buttonStyle(PrimaryButtonStyle())
    }
}

private struct PrimaryButtonStyle: ButtonStyle {
    func makeBody(configuration: Configuration) -> some View {
        configuration.label
            .padding()
    }
}
```

The framework exposes `PrimaryButton`, not every styling helper.

## Initializer Design

Initializers are part of API design.

```swift
public struct AnalyticsConfiguration {
    public let apiKey: String
    public let environment: Environment

    public init(apiKey: String, environment: Environment = .production) {
        self.apiKey = apiKey
        self.environment = environment
    }
}
```

Good initializers:

- Require essential values
- Provide safe defaults
- Avoid exposing internal dependencies
- Use clear argument labels
- Avoid invalid states

## Avoid Leaking DTOs

Bad:

```swift
public struct UserResponse: Decodable {
    public let user_id: String
    public let full_name: String
}
```

If this is public, external modules may depend on backend naming.

Better:

```swift
public struct User {
    public let id: String
    public let name: String

    public init(id: String, name: String) {
        self.id = id
        self.name = name
    }
}

internal struct UserResponse: Decodable {
    let user_id: String
    let full_name: String
}
```

Keep transport details internal.

## Public Protocol Design

```swift
public protocol ImageLoading {
    func image(for url: URL) async throws -> UIImage
}
```

Before exposing a protocol, ask:

- Will external modules implement this?
- Are requirements stable?
- Should this be a concrete type instead?
- Can default implementations reduce future breakage?
- Does it need `Sendable`?

Adding a requirement to a public protocol can break external conformers.

## Open Class Design

Only use `open` when subclassing is supported.

```swift
open class BaseAnalyticsProvider {
    public init() { }

    open func track(event: String, properties: [String: String]) {
        fatalError("Subclasses must override")
    }
}
```

Document:

- Which methods subclasses override
- Which methods they should call `super` from
- Which methods are final
- What thread or actor methods run on

If you do not want subclassing, prefer:

```swift
public final class AnalyticsClient { }
```

## Framework Access Strategy

Useful default strategy:

```text
Start private.
Raise to internal when another file in the module needs it.
Raise to package when another target in the same package needs it.
Raise to public only when external modules need it.
Raise to open only when external subclassing is a designed feature.
```

This avoids accidental API growth.

## API Evolution

Public API changes affect users.

Source-breaking changes:

- Rename public type or method
- Remove public member
- Change argument labels
- Change generic constraints
- Add requirements to public protocols
- Change a public class from open to public
- Remove public initializer

Usually safe changes:

- Add a new public overload
- Add a new public type
- Add defaulted parameters carefully
- Add internal implementation
- Change private/internal code without changing behavior

## Documentation And Examples

Public APIs deserve examples.

```swift
/// Tracks analytics events for the current app session.
public final class AnalyticsClient {
    /// Creates an analytics client.
    public init(configuration: AnalyticsConfiguration) { }
}
```

Good docs explain:

- What the API does
- Main use case
- Thread/actor expectations
- Error behavior
- Lifecycle expectations

## Modern Swift 6.x Framework Notes

Modern framework APIs often need concurrency clarity.

```swift
public protocol ProfileService: Sendable {
    func profile(id: String) async throws -> Profile
}
```

```swift
@MainActor
public final class ProfileViewModel: ObservableObject {
    @Published public private(set) var profile: Profile?
}
```

Think carefully before exposing:

- `@MainActor`
- `Sendable`
- `@Sendable`
- actor types
- nonisolated methods
- async sequences

These choices affect callers.

## iOS Framework Example

```swift
public struct LoginFeatureView: View {
    private let configuration: LoginConfiguration

    public init(configuration: LoginConfiguration) {
        self.configuration = configuration
    }

    public var body: some View {
        LoginRootView(configuration: configuration)
    }
}

public struct LoginConfiguration {
    public let title: String
    public let allowsBiometrics: Bool

    public init(title: String, allowsBiometrics: Bool = true) {
        self.title = title
        self.allowsBiometrics = allowsBiometrics
    }
}

private struct LoginRootView: View {
    let configuration: LoginConfiguration

    var body: some View {
        Text(configuration.title)
    }
}
```

The public API is small and clear.

## Senior Design Checklist

Before making something public, ask:

- Is this part of the product contract?
- Can I support this for multiple releases?
- Does it expose implementation details?
- Can callers create invalid states?
- Should this be configured through a smaller type?
- Should this be `package` instead?
- Should this class be `final`?
- Does concurrency isolation belong in the API?

## Junior Interview Answer

Framework design means exposing only the types and functions other modules need while hiding implementation details.

## Mid-Level Interview Answer

I keep framework public APIs small. I use public for supported entry points, package/internal for shared implementation, private for local details, and open only for intentional subclassing.

## Senior Interview Answer

Framework design is API contract design. Access control protects implementation, reduces coupling, and controls source compatibility. I design public APIs around stable domain concepts, avoid leaking DTOs or infrastructure, prefer final classes unless inheritance is required, and treat concurrency annotations as part of the API surface.

## Points To Remember

- Public API is a promise.
- Open API is an inheritance promise.
- Public initializers must be explicit.
- Public protocols are harder to evolve.
- Keep DTOs and mappers internal.
- Use `package` for shared package implementation.

## Practice

1. Design a public API for a Login framework.
2. Decide which types are public, package, internal, and private.
3. Explain why exposing backend DTOs is risky.
4. Convert an open class design into public final plus protocol composition.
