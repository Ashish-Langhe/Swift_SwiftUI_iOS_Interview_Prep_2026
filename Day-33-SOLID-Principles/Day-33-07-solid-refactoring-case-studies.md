# Day 33: SOLID Refactoring Case Studies

## Why Case Studies Matter

SOLID is easiest to understand through refactoring.

This file shows how to restructure code from messy to maintainable without blindly creating abstractions.

## Case Study 1: Login ViewModel

Bad:

```swift
@MainActor
final class LoginViewModel {
    var email = ""
    var password = ""
    var errorMessage: String?

    func signIn() async {
        guard email.contains("@") else {
            errorMessage = "Invalid email"
            return
        }

        let url = URL(string: "https://api.example.com/login")!
        var request = URLRequest(url: url)
        request.httpMethod = "POST"
        request.httpBody = try? JSONEncoder().encode([
            "email": email,
            "password": password
        ])

        do {
            let data = try await URLSession.shared.data(for: request).0
            let response = try JSONDecoder().decode(LoginResponse.self, from: data)
            KeychainManager.shared.save(response.token)
            AnalyticsManager.shared.track("login_success")
        } catch {
            errorMessage = "Login failed"
        }
    }
}
```

Violations:

- SRP: validation, networking, keychain, analytics, UI state.
- DIP: concrete `URLSession`, `KeychainManager`, `AnalyticsManager`.
- ISP: hidden broad managers.
- OCP: changing auth provider edits ViewModel.

Refactor:

```swift
protocol LoginUseCase {
    func execute(email: String, password: String) async throws
}

struct EmailValidator {
    func isValid(_ email: String) -> Bool {
        email.contains("@") && email.contains(".")
    }
}

@MainActor
final class LoginViewModel {
    var email = ""
    var password = ""
    var state: LoginState = .idle

    private let validator: EmailValidator
    private let loginUseCase: LoginUseCase

    init(validator: EmailValidator, loginUseCase: LoginUseCase) {
        self.validator = validator
        self.loginUseCase = loginUseCase
    }

    var canSubmit: Bool {
        validator.isValid(email) && password.count >= 8
    }

    func signIn() async {
        guard canSubmit else {
            state = .failed("Enter a valid email and password.")
            return
        }

        state = .submitting

        do {
            try await loginUseCase.execute(email: email, password: password)
            state = .signedIn
        } catch {
            state = .failed("Login failed.")
        }
    }
}
```

## Case Study 2: Analytics

Bad:

```swift
FirebaseAnalytics.logEvent("checkout_started", parameters: nil)
```

This leaks vendor API into features.

Better:

```swift
protocol AnalyticsTracking {
    func track(_ event: AnalyticsEvent)
}

struct CheckoutStarted: AnalyticsEvent {
    let cartID: Cart.ID
    var name: String { "checkout_started" }
    var parameters: [String: String] { ["cart_id": cartID.rawValue] }
}
```

Feature:

```swift
analytics.track(CheckoutStarted(cartID: cart.id))
```

Benefits:

- DIP: feature depends on app abstraction.
- OCP: add provider without editing feature.
- SRP: analytics mapping is not in ViewModel.

## Case Study 3: Product Sorting

Bad:

```swift
func sortedProducts(_ products: [Product], sort: SortOption) -> [Product] {
    switch sort {
    case .price:
        products.sorted { $0.price < $1.price }
    case .rating:
        products.sorted { $0.rating > $1.rating }
    case .newest:
        products.sorted { $0.createdAt > $1.createdAt }
    }
}
```

This is fine if options are fixed.

If sorting options come from feature modules, use strategy:

```swift
protocol ProductSorting {
    var title: String { get }
    func sort(_ products: [Product]) -> [Product]
}
```

Decision:

```text
Closed fixed set -> enum switch
Open changing set -> strategy/protocol
```

## Case Study 4: Giant Repository Protocol

Bad:

```swift
protocol ShopRepository {
    func products() async throws -> [Product]
    func orders() async throws -> [Order]
    func user() async throws -> User
    func saveCart(_ cart: Cart) async throws
    func uploadImage(_ image: UIImage) async throws -> URL
}
```

Refactor:

```swift
protocol ProductRepository {
    func products() async throws -> [Product]
}

protocol OrderRepository {
    func orders() async throws -> [Order]
}

protocol CartRepository {
    func saveCart(_ cart: Cart) async throws
}
```

This applies ISP.

## Senior Refactoring Flow

```text
1. Identify pain
2. Identify reasons for change
3. Identify dependencies
4. Extract stable boundaries
5. Add tests around current behavior
6. Refactor one boundary
7. Verify behavior
8. Stop when clarity improves
```

## Reusability Alignment

SOLID supports reuse when:

- Validation is extracted.
- Payment processors are interchangeable.
- Analytics is vendor-independent.
- Repositories expose domain contracts.
- SwiftUI components have focused APIs.

But do not extract for imaginary reuse.

## Common Mistakes

- Refactoring without tests.
- Creating ten abstractions in one pass.
- Making code harder to trace.
- Splitting cohesive behavior.
- Turning SOLID into folder architecture.
- Ignoring performance and simplicity.

## Senior Artifact

```text
Artifact: SOLID Refactoring Plan
Problem:
Current pain:
Principles violated:
Behavior tests:
Extraction step 1:
Extraction step 2:
Risk:
Rollback plan:
Success criteria:
```

## Interview Notes

Junior:

SOLID refactoring separates mixed responsibilities.

Mid-level:

Move networking, validation, persistence, analytics, and navigation into focused types.

Senior:

I refactor toward SOLID by following pain, tests, and change boundaries. I stop when the design becomes clearer and safer, not when every principle has a ceremonial type.

## Practice

1. Refactor a login ViewModel using SOLID.
2. Decide enum vs strategy for sorting.
3. Split a giant repository protocol.
4. Write a refactoring plan before touching code.
