# Day 30: Singleton Pattern

## What Singleton Means

A Singleton provides one shared instance globally.

```swift
final class AnalyticsManager {
    static let shared = AnalyticsManager()

    private init() {}

    func track(_ event: String) {
        // send analytics
    }
}
```

Use:

```swift
AnalyticsManager.shared.track("checkout_tapped")
```

## Why Singleton Is Controversial

Singletons are easy to use but easy to abuse.

They can create:

- Hidden dependencies
- Hard-to-test code
- Global mutable state
- Ordering problems
- Thread-safety issues
- Tight coupling

## Good Singleton Candidates

Singleton may be acceptable for:

- Stateless utility wrappers
- Truly process-wide shared resources
- System-like services with stable behavior
- Lightweight logging entry point

Even then, dependency injection is often better at feature boundaries.

## Bad Singleton Usage

```swift
final class CheckoutViewModel {
    func pay() {
        PaymentManager.shared.charge()
        AnalyticsManager.shared.track("pay")
        SessionManager.shared.refresh()
    }
}
```

This ViewModel hides three dependencies and is hard to test.

Better:

```swift
final class CheckoutViewModel {
    private let paymentService: PaymentService
    private let analytics: AnalyticsClient
    private let session: SessionManaging

    init(
        paymentService: PaymentService,
        analytics: AnalyticsClient,
        session: SessionManaging
    ) {
        self.paymentService = paymentService
        self.analytics = analytics
        self.session = session
    }
}
```

## Singleton With Protocol Boundary

If a singleton exists, hide it behind a protocol at the boundary.

```swift
protocol AnalyticsClient {
    func track(_ event: String)
}

extension AnalyticsManager: AnalyticsClient {}
```

Inject:

```swift
CheckoutViewModel(analytics: AnalyticsManager.shared)
```

Tests can use:

```swift
final class AnalyticsSpy: AnalyticsClient {
    var events: [String] = []

    func track(_ event: String) {
        events.append(event)
    }
}
```

## Thread Safety

Singleton instance creation is safe with `static let`, but its internal mutable state may not be safe.

```swift
final class TokenStore {
    static let shared = TokenStore()
    private var token: String?
}
```

If accessed concurrently, mutable state needs actor, lock, queue, or isolation.

Actor option:

```swift
actor TokenStore {
    static let shared = TokenStore()

    private var token: String?

    func update(_ token: String?) {
        self.token = token
    }
}
```

## Singleton vs Dependency Injection

Singleton:

- Convenient.
- Hidden.
- Harder to test.
- Global lifetime.

DI:

- Explicit.
- Testable.
- Configurable.
- More setup.

Senior preference: use DI for feature code, tolerate singletons only at infrastructure boundaries.

## Common Mistakes

- Singleton for every service.
- Global mutable session state.
- Tests depend on singleton order.
- No reset between tests.
- Singleton directly used inside ViewModels.
- Thread-unsafe mutable state.

## Senior Artifact

```text
Artifact: Singleton Risk Review
Singleton:
Why global:
Mutable state:
Thread safety:
Testability:
Can inject protocol:
Reset behavior:
Alternative:
Decision:
```

## Interview Notes

Junior:

Singleton creates one shared instance.

Mid-level:

Singletons are convenient but can hide dependencies and make tests hard.

Senior:

I avoid singletons in feature logic. If a singleton is unavoidable, I hide it behind a protocol, inject it at boundaries, and review thread safety and test isolation.

## Practice

1. Refactor direct singleton usage into injected protocol.
2. Identify thread-safety risk in a singleton.
3. Create an analytics spy for tests.
4. Explain when singleton is acceptable.
