# Day 24: `@Test`

## What `@Test` Does

`@Test` marks a function as a Swift Testing test.

```swift
import Testing

@Test
func userName_isFormatted() {
    let name = formattedName(first: "Ashish", last: "Langhe")
    #expect(name == "Ashish Langhe")
}
```

`@Test` is an attached macro.

## Test Names

Use readable test names.

```swift
@Test("login succeeds with valid credentials")
func loginSucceeds() async throws {
    #expect(true)
}
```

Swift 6.2 also supports raw identifier display names:

```swift
@Test
func `login succeeds with valid credentials`() async throws {
    #expect(true)
}
```

This can make test names more readable with less extra syntax.

## Arguments

```swift
@Test(arguments: [
    ("", false),
    ("ashish@example.com", true),
])
func emailValidation(input: String, expected: Bool) {
    #expect(isValidEmail(input) == expected)
}
```

Use arguments when the same behavior should be checked across data cases.

## Async And Throwing Tests

```swift
@Test
func profile_loads() async throws {
    let profile = try await service.profile()
    #expect(profile.id == "1")
}
```

Swift Testing naturally supports `async throws`.

## Senior iOS Engineer Artifact

```text
Artifact: Test Case Design
Behavior:
Test name:
Inputs:
Expected output:
Async? throws?
Dependencies:
Test data:
Failure message quality:
```

Senior lens:

- Test names should explain behavior, not implementation.
- Parameterization should keep failures readable.
- Async tests should avoid real timing and real network.
- Tests should be small enough to debug quickly.

## More Coding Examples

### Example 1: Behavior-Oriented Name

```swift
@Test
func `cart total updates after adding item`() {
    var cart = Cart()
    cart.add(CartItem(price: 12))
    #expect(cart.total == 12)
}
```

### Example 2: Parameterized Password Test

```swift
@Test(arguments: [
    ("short", false),
    ("longEnough1", true),
])
func passwordValidation(password: String, expected: Bool) {
    #expect(isValidPassword(password) == expected)
}
```

## Common Mistakes

- Test names that only repeat method names.
- Parameterized tests with too many unrelated cases.
- Real services in unit tests.
- No failure context.
- Testing private helpers directly instead of behavior.

## Interview Guide

Junior:

`@Test` marks a function as a test.

Mid-level:

`@Test` supports names, arguments, async, and throwing tests.

Senior:

I use `@Test` to express behavior clearly. Naming, data, dependency control, and failure readability are all part of test quality.

## Practice

1. Write a named `@Test`.
2. Write an async throwing test.
3. Convert repeated test cases into `@Test(arguments:)`.
4. Improve a vague test name.
