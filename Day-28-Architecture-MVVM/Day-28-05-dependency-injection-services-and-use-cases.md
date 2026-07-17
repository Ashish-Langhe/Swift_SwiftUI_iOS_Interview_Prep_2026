# Day 28: Dependency Injection, Services, And Use Cases

## Why Dependency Injection Matters

Dependency injection means a type receives what it needs instead of creating it internally.

Bad:

```swift
final class LoginViewModel {
    private let authService = LiveAuthService()
}
```

Better:

```swift
final class LoginViewModel {
    private let authService: AuthService

    init(authService: AuthService) {
        self.authService = authService
    }
}
```

This makes the ViewModel testable and flexible.

## Service Protocols

```swift
protocol AuthService {
    func signIn(email: String, password: String) async throws -> UserSession
}
```

Live implementation:

```swift
final class LiveAuthService: AuthService {
    func signIn(email: String, password: String) async throws -> UserSession {
        // URLSession call
    }
}
```

Test implementation:

```swift
struct StubAuthService: AuthService {
    let result: Result<UserSession, Error>

    func signIn(email: String, password: String) async throws -> UserSession {
        try result.get()
    }
}
```

## ViewModel With Dependency

```swift
@Observable
@MainActor
final class LoginViewModel {
    var email = ""
    var password = ""
    var state: LoginState = .idle

    private let authService: AuthService

    init(authService: AuthService) {
        self.authService = authService
    }

    func signIn() async {
        state = .submitting

        do {
            _ = try await authService.signIn(email: email, password: password)
            state = .signedIn
        } catch {
            state = .failed("Invalid email or password.")
        }
    }
}
```

## Use Case Layer

For complex business operations, add a use case.

```swift
protocol SignInUseCase {
    func execute(email: String, password: String) async throws -> UserSession
}
```

Implementation:

```swift
struct LiveSignInUseCase: SignInUseCase {
    let authService: AuthService
    let sessionStore: SessionStore
    let analytics: AnalyticsClient

    func execute(email: String, password: String) async throws -> UserSession {
        let session = try await authService.signIn(email: email, password: password)
        try await sessionStore.save(session)
        analytics.track("sign_in_success")
        return session
    }
}
```

ViewModel:

```swift
private let signInUseCase: SignInUseCase
```

Use cases are helpful when one user action coordinates multiple services.

## When To Add Use Cases

Add use cases when:

- Business action has multiple steps.
- Logic is reused across ViewModels.
- You need transaction-like behavior.
- Testing service coordination matters.
- You want ViewModels smaller.

Skip use cases when:

- The ViewModel only calls one simple service.
- The operation is not reused.
- It adds naming ceremony without clarity.

## Dependency Container

```swift
struct AppDependencies {
    let authService: AuthService
    let productRepository: ProductRepository
    let analytics: AnalyticsClient

    func makeLoginViewModel() -> LoginViewModel {
        LoginViewModel(
            authService: authService
        )
    }
}
```

This centralizes construction without forcing global singletons.

## Environment Injection

Use environment for broad dependencies:

```swift
private struct AnalyticsKey: EnvironmentKey {
    static let defaultValue: AnalyticsClient = NoopAnalyticsClient()
}

extension EnvironmentValues {
    var analytics: AnalyticsClient {
        get { self[AnalyticsKey.self] }
        set { self[AnalyticsKey.self] = newValue }
    }
}
```

Use initializer injection for required feature dependencies.

## Common Mistakes

- ViewModels constructing live services.
- Too many protocols for tiny one-off types.
- Global singleton dependencies.
- Use cases that simply forward one method call.
- Dependency containers that become service locators everywhere.
- Tests using real network/storage.

## Senior iOS Engineer Artifact

```text
Artifact: Dependency Design Review
Feature:
ViewModel dependencies:
Services:
Repositories:
Use cases:
Injected by:
Test doubles:
Global state:
Environment dependencies:
Overengineering risk:
```

## Interview Notes

Junior:

Dependency injection means passing dependencies into an object.

Mid-level:

Inject services into ViewModels so you can test with stubs.

Senior:

I use dependency injection to keep effects outside ViewModels, add use cases when actions coordinate multiple services, avoid singletons, and balance protocols against real testing and modularity needs.

## Practice

1. Replace a live service inside a ViewModel with injected protocol.
2. Add a stub service for tests.
3. Create a use case that coordinates auth and session storage.
4. Explain when a use case is unnecessary.
