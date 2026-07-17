# Day 17: `public`

## What `public` Means

`public` makes a declaration usable from outside the module.

```swift
public struct APIClient {
    public init() { }

    public func fetchUser(id: String) async throws -> User {
        User(id: id, name: "Ashish")
    }
}
```

Another module can import your module and use `APIClient`.

## Beginner Mental Model

Use `public` when you want:

```text
Other modules are allowed to use this, but not subclass or override it unless I allow that separately.
```

`public` exposes usage. `open` exposes subclassing/overriding.

## Public Initializers Are Not Automatic

A public type does not automatically get a public initializer.

```swift
public struct User {
    public let id: String
    public let name: String
}
```

The memberwise initializer is not public by default.

If external modules should create `User`, write a public initializer:

```swift
public struct User {
    public let id: String
    public let name: String

    public init(id: String, name: String) {
        self.id = id
        self.name = name
    }
}
```

This is a common interview trap.

## Public Members Are Not Automatic

If a type is public, its members are still internal by default unless marked public.

```swift
public struct Profile {
    let id: String // internal, not accessible outside module
}
```

Better:

```swift
public struct Profile {
    public let id: String

    public init(id: String) {
        self.id = id
    }
}
```

## Real iOS Use Cases

Use `public` for:

- SDK APIs
- Design system components consumed by app modules
- Feature entry views exposed to the app shell
- Public package products
- Domain models intentionally shared across modules
- Protocols external modules should conform to or depend on

```swift
public struct AppPrimaryButton: View {
    private let title: String
    private let action: () -> Void

    public init(_ title: String, action: @escaping () -> Void) {
        self.title = title
        self.action = action
    }

    public var body: some View {
        Button(title, action: action)
    }
}
```

The component is public. Its stored properties remain private.

## Public API Is A Promise

Once a declaration is public, external code can depend on it.

Changing it later can be a breaking change.

Breaking changes include:

- Renaming a public type
- Removing a public method
- Changing parameter labels
- Changing return types
- Making a public declaration internal
- Changing behavior in a surprising way

```swift
public func loadUser(id: String) async throws -> User
```

Changing this to:

```swift
public func loadUser(userID: String) async throws -> User
```

can break callers because argument labels are part of the call site.

## Public Protocols

Public protocols should be designed carefully.

```swift
public protocol ImageLoading {
    func image(for url: URL) async throws -> UIImage
}
```

Before making a protocol public, ask:

- Do external modules need to conform to it?
- Are the requirements stable?
- Will adding new requirements later break conformers?
- Should this be internal or package instead?

Adding a requirement to a public protocol can break external conforming types unless you provide a default implementation.

## Public And Concurrency

Modern public APIs often include concurrency annotations.

```swift
public protocol UserRepository: Sendable {
    func user(id: String) async throws -> User
}
```

These annotations become part of the API contract.

Be intentional with:

- `async`
- `throws`
- `Sendable`
- `@MainActor`
- actor types
- escaping closure sendability

```swift
@MainActor
public final class ProfileViewModel: ObservableObject {
    @Published public private(set) var name = ""
}
```

External callers now know this view model is main-actor isolated.

## Public Type, Private Implementation

Good public API hides implementation.

```swift
public final class AnalyticsClient {
    private let transport: AnalyticsTransport

    public init(apiKey: String) {
        self.transport = AnalyticsTransport(apiKey: apiKey)
    }

    public func track(_ event: AnalyticsEvent) {
        transport.send(event)
    }
}

private final class AnalyticsTransport {
    let apiKey: String

    init(apiKey: String) {
        self.apiKey = apiKey
    }

    func send(_ event: AnalyticsEvent) { }
}
```

Consumers see a small API.

## Advanced Design Notes

Senior engineers treat public API like product design.

Public API should be:

- Small
- Stable
- Intention-revealing
- Hard to misuse
- Documented
- Testable
- Compatible with future evolution

Avoid exposing:

- DTOs that mirror backend responses
- Internal caches
- Low-level transport types
- Helpers used only by one implementation
- Mutable state that external code should not control

## Modern Swift 6.x Notes

`public` remains stable in Swift 6.x.

Modern Swift makes public API design more sensitive because concurrency and sendability can be visible to callers.

For example:

```swift
public func observeChanges(_ handler: @escaping @Sendable (Change) -> Void)
```

The `@Sendable` requirement affects what callers can capture safely.

## Junior Interview Answer

`public` makes a declaration accessible from other modules.

## Mid-Level Interview Answer

A public type's members are not automatically public. If another module needs to initialize or read members, the initializer and members must also be public.

## Senior Interview Answer

Public API is an external contract. I keep it small and stable, hide implementation with private/internal/package declarations, and think carefully before exposing protocols, initializers, argument labels, and concurrency annotations because they affect source compatibility.

## Points To Remember

- `public` allows use from other modules.
- Public classes cannot be subclassed outside the module unless `open`.
- Public methods cannot be overridden outside the module unless `open`.
- Public type members are internal by default.
- Public initializers must be written explicitly when needed.

## Practice

1. Create a public design system button with private implementation details.
2. Explain why a public struct may still be impossible to initialize from another module.
3. Identify breaking changes in a public function signature.
4. Design a public protocol and explain the risk of adding requirements later.
