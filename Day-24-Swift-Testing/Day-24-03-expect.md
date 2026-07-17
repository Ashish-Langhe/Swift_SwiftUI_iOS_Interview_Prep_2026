# Day 24: `#expect`

## What `#expect` Does

`#expect` checks that an expression is true.

```swift
#expect(cart.total == 25)
```

It is a freestanding expression macro.

Because it sees the source expression, it can provide rich failure details.

## Basic Usage

```swift
@Test
func user_isAdult() {
    let user = User(age: 21)
    #expect(user.age >= 18)
}
```

## Multiple Expectations

```swift
@Test
func profile_mapping() {
    let viewModel = ProfileViewModel(profile: profile)

    #expect(viewModel.title == "Ashish")
    #expect(viewModel.subtitle == "iOS Engineer")
}
```

Keep multiple expectations related to one behavior.

## `#require`

Use `#require` when the test cannot continue without a value.

```swift
let profile = try #require(await service.profile())
#expect(profile.name == "Ashish")
```

## Throws Expectations

Swift Testing supports expressive throw checking.

```swift
#expect(throws: LoginError.invalidCredentials) {
    try login(email: "", password: "")
}
```

Swift 6.1 refined throwing expectations to make caught errors easier to inspect.

## Senior iOS Engineer Artifact

```text
Artifact: Expectation Quality Review
Behavior:
Expectation:
Failure output:
Should test continue after failure?
Use #expect or #require?
Error expectation:
Additional attachment/log needed:
```

Senior lens:

- Expectations should describe business behavior.
- Use `#require` for preconditions.
- Avoid overly broad expectations.
- Do not hide important setup failure.

## More Coding Examples

### Example 1: Optional Requirement

```swift
@Test
func fixture_decodesUser() throws {
    let url = try #require(Bundle.module.url(forResource: "user", withExtension: "json"))
    let data = try Data(contentsOf: url)
    let user = try JSONDecoder().decode(User.self, from: data)
    #expect(user.id == "1")
}
```

### Example 2: Error Check

```swift
@Test
func emptyPassword_throwsValidationError() {
    #expect(throws: ValidationError.emptyPassword) {
        try validate(password: "")
    }
}
```

## Common Mistakes

- Using `#expect` where `#require` is needed.
- Checking too many unrelated things in one test.
- Expecting implementation details.
- Ignoring thrown error type.
- Writing expectations that are hard to diagnose.

## Interview Guide

Junior:

`#expect` checks a test condition.

Mid-level:

Use `#expect` for assertions and `#require` when the test cannot continue without a value.

Senior:

I design expectations for useful failure output. A failed test should explain the broken behavior quickly.

## Practice

1. Replace a boolean assert with `#expect`.
2. Use `#require` for a fixture URL.
3. Check a thrown validation error.
4. Split unrelated expectations into separate tests.
