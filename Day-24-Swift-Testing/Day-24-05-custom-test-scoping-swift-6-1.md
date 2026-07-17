# Day 24: Custom Test Scoping From Swift 6.1

## What Custom Test Scoping Is

Swift 6.1 added custom Swift Testing traits that can perform logic before or after tests run.

This is done with `TestScoping`.

Use cases:

- Bind task-local values.
- Provide mock credentials.
- Set temporary environment.
- Create scoped resources.
- Run setup/teardown around a test.

## Conceptual Example

```swift
import Testing

struct MockAPITrait: TestTrait, TestScoping {
    func provideScope(
        for test: Test,
        testCase: Test.Case?,
        performing function: @Sendable () async throws -> Void
    ) async throws {
        try await APIEnvironment.$current.withValue(.mock) {
            try await function()
        }
    }
}
```

Then expose it:

```swift
extension Trait where Self == MockAPITrait {
    static var mockAPI: Self { Self() }
}
```

Usage:

```swift
@Test(.mockAPI)
func profile_usesMockEnvironment() async throws {
    #expect(APIEnvironment.current == .mock)
}
```

## Why This Matters

Before custom scoping, setup/teardown could become repeated or global.

Scoped traits keep setup:

- Explicit
- Local to tests
- Async-aware
- Reusable
- Less global-state heavy

## Real iOS Use Case

Testing API clients that read task-local credentials:

```swift
enum APICredentials {
    @TaskLocal static var current: String = "production"
}

struct MockCredentialsTrait: TestTrait, TestScoping {
    func provideScope(
        for test: Test,
        testCase: Test.Case?,
        performing function: @Sendable () async throws -> Void
    ) async throws {
        try await APICredentials.$current.withValue("mock-key") {
            try await function()
        }
    }
}
```

## Senior iOS Engineer Artifact

```text
Artifact: Custom Trait Scoping Review
Trait:
Scoped resource:
Setup:
Teardown:
Async safety:
Global state avoided:
Failure behavior:
Tests using it:
```

Senior lens:

- Scoping traits should make setup clearer, not hidden.
- Avoid global mutable test state.
- Prefer task-local scoped context for async tests.
- Document what the trait changes.

## More Coding Examples

### Example 1: Scoped Locale

```swift
struct LocaleTrait: TestTrait, TestScoping {
    func provideScope(
        for test: Test,
        testCase: Test.Case?,
        performing function: @Sendable () async throws -> Void
    ) async throws {
        try await AppLocale.$current.withValue(Locale(identifier: "en_US")) {
            try await function()
        }
    }
}
```

### Example 2: Use Scoped Trait

```swift
@Test(.mockAPI)
func request_usesMockCredentials() async throws {
    #expect(APICredentials.current == "mock-key")
}
```

## Common Mistakes

- Hiding too much setup in traits.
- Using global mutable state.
- Not documenting scoped behavior.
- Sharing state between tests accidentally.
- Making custom traits when helper functions are enough.

## Interview Guide

Junior:

Custom test scoping lets a trait run setup around a test.

Mid-level:

Swift 6.1 added `TestScoping` so custom traits can provide async setup/teardown behavior.

Senior:

I use custom scoping for reusable async-safe context, especially task-local values, while keeping behavior explicit and isolated.

## Practice

1. Design a mock API credentials trait.
2. Explain task-local scoping.
3. Compare custom trait vs helper setup function.
4. Identify hidden setup risks.
