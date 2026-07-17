# Day 24: Traits

## What Traits Are

Traits customize test behavior.

Swift Testing uses traits in `@Test` and suites.

Conceptually, traits can express:

- Tags
- Conditions
- Time limits
- Bug references
- Custom behavior
- Setup/teardown scopes

```swift
@Test(.tags(.networking))
func endpoint_buildsCorrectPath() {
    #expect(true)
}
```

## Why Traits Matter

Traits make test metadata explicit.

They help with:

- Organization
- Filtering
- Conditional execution
- Shared behavior
- Documentation
- CI grouping

## Real iOS Use Case

```swift
@Test(.tags(.payments))
func paymentRequest_containsIdempotencyKey() {
    let request = makePaymentRequest()
    #expect(request.headers["Idempotency-Key"] != nil)
}
```

Tags make it easier to identify critical test areas.

## Suite-Level Traits

Traits can apply to groups.

```swift
@Suite(.tags(.profile))
struct ProfileMappingTests {
    @Test
    func mapsDisplayName() {
        #expect(true)
    }
}
```

## Senior iOS Engineer Artifact

```text
Artifact: Test Trait Strategy
Trait:
Scope: test / suite
Purpose:
CI use:
Filtering need:
Setup/teardown effect:
Owner:
```

Senior lens:

- Traits are test metadata and behavior.
- Avoid tag explosion.
- Use traits to improve CI and clarity.
- Custom traits should be worth their complexity.

## More Coding Examples

### Example 1: Tag Critical Tests

```swift
@Test(.tags(.critical))
func loginToken_isStoredSecurely() {
    #expect(true)
}
```

### Example 2: Suite Tag

```swift
@Suite(.tags(.checkout))
struct CheckoutTests {
    @Test
    func totalIncludesTax() {
        #expect(true)
    }
}
```

## Common Mistakes

- Too many vague tags.
- Traits that hide test setup.
- Custom traits with unclear behavior.
- CI filters that skip important tests accidentally.
- Using traits instead of clear test names.

## Interview Guide

Junior:

Traits customize or label tests.

Mid-level:

Use traits for tags, conditions, and shared test metadata.

Senior:

Traits should improve test organization and CI behavior without making tests magical.

## Practice

1. Add a tag trait to a test.
2. Add a suite-level trait.
3. Design tags for critical payment tests.
4. Explain why too many traits can hurt readability.
