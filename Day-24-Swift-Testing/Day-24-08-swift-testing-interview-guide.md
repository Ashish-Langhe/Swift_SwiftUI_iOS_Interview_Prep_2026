# Day 24: Swift Testing Interview Guide

## One-Minute Senior Answer

Swift Testing is Swift's modern testing framework introduced with Swift 6. It uses macros like `@Test`, `@Suite`, `#expect`, and `#require` to write expressive tests with rich failure output. Swift 6.1 added custom test scoping traits through `TestScoping`, and Swift 6.2 added exit testing, attachments, and raw identifier display names. A senior iOS engineer uses Swift Testing to create deterministic, async-safe, evidence-rich tests that validate behavior rather than implementation details.

## Junior Questions

What is `@Test`?

An attached macro that marks a function as a test.

What is `#expect`?

A macro that checks an expectation and reports rich failure details.

Can Swift Testing handle async tests?

Yes. Test functions can be `async throws`.

## Mid-Level Questions

When use `#require`?

Use `#require` when the test cannot continue without a value.

What are traits?

Traits add metadata or behavior to tests and suites.

What are parameterized tests?

Tests that run the same logic with multiple input arguments.

## Senior Questions

How do you design good Swift tests?

I test behavior, inject dependencies, control time/network/file system, keep failure output useful, and use parameterization only when it improves clarity.

What did Swift 6.1 add?

Custom test scoping traits with `TestScoping`, plus improved throwing expectations.

What did Swift 6.2 add?

Exit testing, attachments, and raw identifier display names.

## Senior iOS Engineer Artifact

```text
Artifact: Swift Testing Suite Review
Feature:
Behavior under test:
Async dependencies:
Fixtures:
Parameterized cases:
Traits:
Custom scoping:
Exit tests:
Attachments:
CI behavior:
Flakiness risk:
```

## Real iOS Scenario

Testing a profile feature:

```swift
@Suite(.tags(.profile))
struct ProfileFeatureTests {
    @Test
    func `loading profile displays full name`() async throws {
        let service = StubProfileService(profile: .init(id: "1", name: "Ashish"))
        let viewModel = await ProfileViewModel(service: service)

        await viewModel.load()

        await #expect(viewModel.title == "Ashish")
    }
}
```

Senior discussion:

- Dependency is stubbed.
- Test is async-safe.
- Name describes behavior.
- No network.
- Failure output points to user-visible state.

## Latest Swift Notes

- Swift 6 introduced Swift Testing.
- Swift 6.1 added custom test scoping traits using `TestScoping`.
- Swift 6.1 improved `#expect(throws:)` / `#require(throws:)` ergonomics.
- Swift 6.2 added exit testing.
- Swift 6.2 added attachments.
- Swift 6.2 added raw identifier display names for tests and suites.

## Common Traps

- Testing private implementation details.
- Real network calls in tests.
- Time-dependent flaky async tests.
- Overusing parameterized tests.
- Hiding setup in custom traits.
- Attaching sensitive data.
- Using exit tests for user errors.

## More Coding Examples

### Example 1: Parameterized Validation

```swift
@Test(arguments: [
    ("", false),
    ("ashish@example.com", true),
])
func emailValidation(input: String, expected: Bool) {
    #expect(isValidEmail(input) == expected)
}
```

### Example 2: Throwing Expectation

```swift
@Test
func invalidLogin_throws() {
    #expect(throws: LoginError.invalidCredentials) {
        try validateLogin(email: "", password: "")
    }
}
```

### Example 3: Scoped Mock Context

```swift
@Test(.mockAPI)
func request_usesMockCredentials() async throws {
    #expect(APICredentials.current == "mock-key")
}
```

## Interview Closing Answer

At senior level, testing is not just checking values. It is designing fast feedback. Swift Testing gives expressive macros, async support, traits, scoped setup, exit tests, and attachments. I use those tools to make failures easy to understand and behavior safe to change.

## Practice Prompts

1. Write a Swift Testing suite for login validation.
2. Add parameterized cases.
3. Add a custom scoping trait for mock credentials.
4. Decide whether a failure should throw or exit.
5. Add an attachment plan for JSON decoding failures.
