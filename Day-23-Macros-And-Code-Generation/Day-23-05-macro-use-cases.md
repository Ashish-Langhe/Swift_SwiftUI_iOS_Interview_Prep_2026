# Day 23: Macro Use Cases

## Common Macro Use Cases

Macros are useful when code is:

- Repetitive
- Pattern-based
- Compile-time validateable
- Source-aware
- Hard to maintain by hand

Good macro use cases:

- Testing assertions
- Mock generation
- Debug descriptions
- Observable models
- API endpoint declarations
- Localization key validation
- Feature flag declarations
- Codable helpers

## Use Case: Test Assertions

```swift
#expect(order.total == 99)
```

The macro can capture both sides of the expression and show rich failure output.

## Use Case: Mocks

```swift
@Mockable
protocol ProductLoading {
    func products() async throws -> [Product]
}
```

Generated mock could help tests avoid boilerplate.

## Use Case: Localization

```swift
let title = #localized("profile.title")
```

A macro can validate that the key exists.

## Use Case: Feature Flags

```swift
@FeatureFlag("new_checkout")
var isNewCheckoutEnabled: Bool
```

Generated code can keep keys, defaults, and debug metadata consistent.

## Use Case: API Endpoints

```swift
@Endpoint("GET /users/{id}")
struct GetUser {
    let id: String
}
```

Potential generation:

- Path builder
- HTTP method
- request model
- response typing

## Senior iOS Engineer Artifact

```text
Artifact: Macro Use Case Evaluation
Use case:
Manual boilerplate today:
Compile-time validation possible:
Generated code:
Developer ergonomics benefit:
Debugging cost:
Build cost:
Alternative:
```

Senior lens:

- Macros should improve correctness or ergonomics meaningfully.
- Avoid macro magic for business logic.
- Prefer simple Swift abstractions when they solve the problem.
- Macro APIs need documentation and examples.

## More Coding Examples

### Example 1: Plain Swift Before Macro

```swift
enum L10n {
    static let profileTitle = String(localized: "profile.title")
}
```

This may be enough for many apps.

### Example 2: Macro When Validation Matters

```swift
let title = #localized("profile.title")
```

This is attractive if the macro validates keys at compile time.

## Common Mistakes

- Macro for every pattern.
- Hiding network/business behavior in generated code.
- Not exposing generated output to reviewers.
- No tests for bad macro usage.
- Using macros when a protocol extension would work.

## Interview Guide

Junior:

Macros reduce repeated code by generating Swift at compile time.

Mid-level:

Good use cases include assertions, mock generation, localization validation, and repeated declarations.

Senior:

I evaluate macros by correctness benefit, diagnostics, generated API, build cost, and whether simpler Swift would be clearer.

## Practice

1. Pick one repeated pattern in an app.
2. Solve it without macros.
3. Decide whether a macro adds enough value.
4. Write bad-usage test cases for the macro.
