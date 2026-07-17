# Day 17: `package`

## What `package` Means

`package` allows a declaration to be used by any target inside the same Swift package, but not by code outside that package.

It sits between `internal` and `public`.

```text
private < fileprivate < internal < package < public < open
```

```swift
package struct FeatureFlag {
    package let name: String
    package let isEnabled: Bool
}
```

Targets in the same package can use `FeatureFlag`. External package users cannot.

## Why `package` Exists

Before `package`, teams often had an awkward choice:

- Use `internal`: visible only inside one target.
- Use `public`: visible to everyone importing the package.

But real packages often have multiple targets that need to share implementation.

```text
MyAppFeatures package
  LoginFeature target
  ProfileFeature target
  SharedFeatureSupport target
  FeatureTestingSupport target
```

Some APIs should be shared across these targets but hidden from app/framework consumers.

That is what `package` is for.

## Beginner Mental Model

Use `package` when you want:

```text
This is shared inside the package, but it is not public product API.
```

## Swift Package Example

Package layout:

```text
Sources/
  Networking/
    APIClient.swift
  NetworkingSupport/
    RequestBuilder.swift
  AppFeature/
    FeatureService.swift
```

`RequestBuilder` can be package-visible:

```swift
package struct RequestBuilder {
    package func makeRequest(path: String) -> URLRequest {
        URLRequest(url: URL(string: "https://api.example.com\(path)")!)
    }
}
```

Other targets in the same package can use it, but outside apps cannot.

## Real iOS Use Cases

Use `package` for:

- Shared feature infrastructure inside one package
- Internal design system primitives
- Test support utilities used by multiple package targets
- Mock factories shared across package tests
- Shared routing contracts not intended for external consumers
- DTO mappers used by multiple targets in the package

```swift
package enum AnalyticsEventName {
    package static let checkoutStarted = "checkout_started"
    package static let checkoutCompleted = "checkout_completed"
}
```

Your feature targets can reuse event names without making them part of your public SDK.

## `package` vs `internal`

`internal` is target-level.

`package` is package-level.

```text
Same target: internal works
Different target, same package: package works
Different package: public/open required
```

Example:

```swift
// Target: DesignSystemCore
package struct SpacingScale {
    package let small: CGFloat = 8
    package let medium: CGFloat = 16
}
```

Another target in the same package can use `SpacingScale`.

An app importing the published design system product cannot use it unless it is `public`.

## `package` vs `public`

Use `public` for supported API.

Use `package` for shared implementation.

```swift
public struct PrimaryButton: View {
    private let title: String

    public init(_ title: String) {
        self.title = title
    }

    public var body: some View {
        Text(title)
            .padding(SpacingScale.defaultPadding)
    }
}

package enum SpacingScale {
    package static let defaultPadding: CGFloat = 16
}
```

The button is API. The spacing implementation can remain package-only.

## Package Access And Testing

`package` is useful for test support targets.

```swift
package struct UserFactory {
    package static func makeUser(id: String = "1") -> User {
        User(id: id, name: "Test User")
    }
}
```

Multiple test targets inside the package can reuse this without making it public.

This avoids polluting production API just for tests.

## Framework Design With `package`

If you ship a framework built from a package, `package` helps keep the external API small.

```text
Public API:
  LoginView
  LoginConfiguration

Package API:
  LoginValidator
  LoginAnalyticsMapper
  LoginPreviewData
```

This is cleaner than making everything public.

## Advanced Rules And Constraints

Access levels compose with the types they mention.

A `public` declaration should not expose a `package` type in its signature.

```swift
package struct InternalConfiguration { }

public struct LoginView {
    // Invalid design: public API exposes package-only type.
    // public init(configuration: InternalConfiguration) { }
}
```

Better:

```swift
public struct LoginConfiguration {
    public let allowsBiometrics: Bool

    public init(allowsBiometrics: Bool) {
        self.allowsBiometrics = allowsBiometrics
    }
}
```

## Modern Swift 6.x Notes

`package` was introduced before Swift 6 and remains important in modern Swift package design.

Current practical relevance:

- SwiftPM-heavy apps benefit from package-level sharing.
- Large iOS codebases can avoid making support APIs public.
- Package access works well with multi-target packages.
- Concurrency annotations on public APIs should be intentional, so keeping helper APIs package-only reduces long-term compatibility pressure.

## Common Mistakes

Mistake:

Making helper APIs public because another target needs them.

Better:

Use `package` if both targets are inside the same package.

Mistake:

Using `package` in a single-target app and expecting a benefit.

Better:

Use `internal` or `private` unless you actually have package-level target sharing.

## Junior Interview Answer

`package` means a declaration is accessible across targets in the same Swift package, but not outside the package.

## Mid-Level Interview Answer

`package` is useful when a Swift package has multiple targets that need to share code without exposing that code as public API.

## Senior Interview Answer

`package` is an API design tool for multi-target Swift packages. It prevents public API pollution while allowing shared internal infrastructure across package targets. I use it for support types, test factories, shared feature infrastructure, and implementation details that cross target boundaries but should not become external contract.

## Points To Remember

- `package` is broader than `internal`.
- `package` is narrower than `public`.
- It is mainly useful in Swift Package Manager packages.
- It helps avoid accidental public API.
- Public APIs should not expose package-only types.

## Practice

1. Design a package with `Networking`, `FeatureA`, and `FeatureSupport` targets.
2. Mark shared implementation as `package` and external API as `public`.
3. Explain why `package` is better than `public` for test factories.
4. Identify a public helper that should really be package-only.
