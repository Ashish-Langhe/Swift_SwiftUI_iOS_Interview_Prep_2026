# Day 23: Freestanding Macros

## What Freestanding Macros Are

Freestanding macros appear as standalone expressions or declarations.

They start with `#`.

Examples:

```swift
#expect(total == 10)
#warning("Review this migration")
```

Freestanding macros can produce expressions, declarations, or diagnostics depending on their role.

## Expression Macros

Expression macros expand into an expression.

Swift Testing's `#expect` is a familiar example:

```swift
#expect(user.isActive)
```

It captures expression details so failures are more useful than a plain boolean assert.

## Declaration Macros

A freestanding declaration macro can produce declarations.

Conceptual example:

```swift
#GenerateFeatureFlags("FeatureFlags.json")
```

This could generate Swift declarations from a config file.

## Realistic App Example

Imagine a localization macro:

```swift
let title = #localized("settings.notifications.title")
```

Potential benefits:

- Validate key exists at compile time.
- Generate type-safe access.
- Avoid typo-prone string keys.

Senior caution:

This must integrate with localization tooling and produce clear diagnostics.

## Macro Declaration Shape

Macro declarations usually expose the macro API:

```swift
@freestanding(expression)
public macro localized(_ key: String) -> String =
    #externalMacro(module: "LocalizationMacros", type: "LocalizedMacro")
```

The implementation lives in a macro target.

## Senior iOS Engineer Artifact

```text
Artifact: Freestanding Macro Design
Macro syntax:
Input:
Output expression/declaration:
Validation:
Diagnostics:
Fallback if macro fails:
Build impact:
Test cases:
```

Senior lens:

- Freestanding macros should make call sites clearer.
- Inputs should be easy to validate.
- Diagnostics are part of the user experience.
- Generated code should be inspectable through macro expansion tooling.

## More Coding Examples

### Example 1: Testing Expression Macro

```swift
#expect(viewModel.state == .loaded)
```

This gives better failure output than a plain `assert`.

### Example 2: Compile-Time Warning

```swift
#warning("Remove temporary migration path before release")
```

Freestanding diagnostics can communicate migration work directly in code.

## Common Mistakes

- Using a macro where a function is clearer.
- Accepting arbitrary strings without validation.
- Producing diagnostics that point to the wrong source location.
- Hiding generated declarations from documentation.

## Interview Guide

Junior:

Freestanding macros start with `#` and are used as expressions or declarations.

Mid-level:

They are useful for compile-time validation, diagnostics, and source-aware generated code.

Senior:

I evaluate freestanding macros by call-site clarity, validation power, diagnostics quality, and generated-code transparency.

## Practice

1. Explain why `#expect` can show better failures than `assert`.
2. Design a `#localized` macro.
3. List diagnostics a localization macro should produce.
4. Decide whether a helper function would be enough.
