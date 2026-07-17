# Day 24: Swift Testing Basics

## What Swift Testing Is

Swift Testing is Swift's modern testing library introduced with Swift 6.

It uses macros such as:

- `@Test`
- `@Suite`
- `#expect`
- `#require`

Basic test:

```swift
import Testing

@Test
func total_isCalculatedCorrectly() {
    let total = 2 + 3
    #expect(total == 5)
}
```

## Why Swift Testing Matters

Swift Testing is designed for Swift-first testing:

- Expressive assertions
- Async test support
- Parameterized tests
- Traits
- Rich failure output
- Cross-platform support
- Integration with SwiftPM

## XCTest vs Swift Testing

XCTest is still widely used in iOS projects.

Swift Testing provides a more Swift-native style.

```swift
#expect(user.name == "Ashish")
```

The macro captures expression details and can show richer failure information.

## Async Tests

```swift
@Test
func profile_loadsName() async throws {
    let service = StubProfileService()
    let profile = try await service.profile()
    #expect(profile.name == "Ashish")
}
```

Async tests are first-class.

## Parameterized Tests

```swift
@Test(arguments: [
    ("", false),
    ("a@b.com", true),
])
func emailValidation(input: String, expected: Bool) {
    #expect(isValidEmail(input) == expected)
}
```

This reduces repeated test functions.

## Senior iOS Engineer Artifact

```text
Artifact: Swift Testing Strategy
Feature:
Unit tests:
Async tests:
Parameterized cases:
Traits needed:
Fixtures:
Failure evidence:
CI behavior:
XCTest coexistence:
```

Senior lens:

- Tests are design feedback.
- Prefer small deterministic tests.
- Async tests should control dependencies and time.
- Parameterized tests should improve clarity, not hide intent.

## More Coding Examples

### Example 1: Simple Domain Test

```swift
@Test
func cart_totalIncludesAllItems() {
    let cart = Cart(items: [
        CartItem(price: 10),
        CartItem(price: 15),
    ])

    #expect(cart.total == 25)
}
```

### Example 2: Async ViewModel Test

```swift
@Test
func loading_setsLoadedState() async throws {
    let viewModel = await ProductsViewModel(service: StubProductsService())
    await viewModel.load()
    await #expect(viewModel.state == .loaded)
}
```

## Common Mistakes

- Testing implementation details instead of behavior.
- Using real network calls.
- Not controlling time in async tests.
- Making parameterized tests too broad.
- Mixing XCTest and Swift Testing without a migration plan.

## Interview Guide

Junior:

Swift Testing is a Swift-first testing library that uses `@Test` and `#expect`.

Mid-level:

It supports async tests, parameterized tests, and expressive failure output.

Senior:

I design Swift Testing suites around deterministic behavior, dependency injection, useful failure evidence, and CI reliability.

## Practice

1. Write a simple `@Test`.
2. Add an async test.
3. Convert repeated tests into parameterized tests.
4. Explain when XCTest may still exist in a project.
