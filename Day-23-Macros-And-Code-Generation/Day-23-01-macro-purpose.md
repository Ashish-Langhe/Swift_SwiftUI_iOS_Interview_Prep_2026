# Day 23: Macro Purpose

## What Macros Are

Macros generate Swift code at compile time.

They let library authors reduce repetitive boilerplate while keeping generated code type-checked by the compiler.

Swift macros are used by:

- Swift Testing (`@Test`, `#expect`)
- Observation (`@Observable`)
- Debugging support (`@DebugDescription`)
- Custom code generation libraries
- DSL-style APIs

## Why Macros Exist

Macros solve repetition that is too mechanical for humans but still needs compile-time safety.

Examples:

- Generate conformance code.
- Generate test declarations.
- Generate debug descriptions.
- Validate expressions.
- Create domain-specific APIs.

Without macros, teams often rely on:

- Hand-written boilerplate
- Scripts
- Sourcery-style generation
- Runtime reflection

Macros bring some of that generation into Swift's compiler pipeline.

## Compile-Time vs Runtime

Macro expansion happens during compilation.

```swift
#expect(user.name == "Ashish")
```

The macro can capture source-level expression details and produce helpful failure output.

Runtime reflection cannot easily know the original source expression.

## Real iOS Use Case

Imagine a feature flag declaration:

```swift
@FeatureFlag("new_checkout")
var isNewCheckoutEnabled: Bool
```

A macro could generate:

- Storage key
- Default value access
- Analytics metadata
- Debug UI label

Senior note:

The generated API must still be easy to understand and debug.

## When Not To Use Macros

Avoid macros when:

- A function is enough.
- A property wrapper is enough.
- A protocol extension is enough.
- Code generation hides business rules.
- Compile errors become hard to understand.
- Build-time cost is not worth it.

## Senior iOS Engineer Artifact

```text
Artifact: Macro Justification Review
Macro:
Problem solved:
Boilerplate removed:
Generated code shape:
Debuggability:
Build impact:
Alternative considered:
API stability:
```

Senior lens:

- Macros are API design, not magic.
- Generated code should be predictable.
- Macro errors must help users fix mistakes.
- Build performance and CI impact matter.

## More Coding Examples

### Example 1: Macro-Like Motivation Without Macro

```swift
struct AnalyticsEvent {
    let name: String
    let properties: [String: String]
}

let event = AnalyticsEvent(name: "checkout_started", properties: [:])
```

If this pattern repeats hundreds of times with metadata, a macro may become useful.

### Example 2: Better First Step

```swift
extension AnalyticsEvent {
    static let checkoutStarted = AnalyticsEvent(name: "checkout_started", properties: [:])
}
```

Before writing a macro, try plain Swift. Reach for macros when plain Swift becomes repetitive or less safe.

## Common Mistakes

- Using macros for simple helper logic.
- Hiding generated behavior from users.
- Creating poor diagnostics.
- Making builds slower for small convenience.
- Generating public API without stability review.

## Interview Guide

Junior:

Macros generate Swift code at compile time.

Mid-level:

They reduce boilerplate while preserving type checking and source-level diagnostics.

Senior:

Macros are compile-time API tools. I use them only when they improve safety or ergonomics enough to justify build, debugging, and maintenance costs.

## Practice

1. Identify boilerplate in a feature.
2. Solve it with plain Swift first.
3. Decide whether a macro is justified.
4. Write a macro design review artifact.
