# Day 22: Warning Control From Swift 6.2

## What Warning Control Means

Swift 6.2 introduced precise warning control in SwiftPM package manifests.

You can configure warnings by diagnostic group using:

- `SwiftSetting.treatAllWarnings`
- `SwiftSetting.treatWarning`

This lets packages promote, downgrade, or manage warning categories with more precision.

## Treat All Warnings As Errors

```swift
.target(
    name: "Core",
    swiftSettings: [
        .treatAllWarnings(as: .error),
    ]
)
```

This is useful for strict modules where warnings should fail CI.

## Override One Warning Group

Swift 6.2 lets you control a diagnostic group by name.

```swift
.target(
    name: "Core",
    swiftSettings: [
        .treatAllWarnings(as: .error),
        .treatWarning("DeprecatedDeclaration", as: .warning),
    ]
)
```

This promotes warnings to errors except deprecation warnings, which remain warnings.

## Why This Matters

Before precise warning control, teams often had all-or-nothing choices.

Senior teams need more nuance:

- New code should be strict.
- Legacy deprecations may need gradual cleanup.
- Generated code may need different policy.
- CI should catch important warnings.
- Migration periods should be controlled.

## Real iOS Use Case

```swift
.target(
    name: "PaymentCore",
    swiftSettings: [
        .treatAllWarnings(as: .error),
    ]
),
.target(
    name: "LegacyPaymentAdapter",
    swiftSettings: [
        .treatAllWarnings(as: .warning),
    ]
)
```

New core code is strict. Legacy adapter code can migrate gradually.

## Senior iOS Engineer Artifact

```text
Artifact: Warning Policy
Target:
Default warning behavior:
Warnings promoted to errors:
Warnings downgraded:
Reason:
CI enforcement:
Migration plan:
Owner:
Review date:
```

Senior lens:

- Warning control is engineering policy.
- Strictness should be target-specific when needed.
- Downgrades need a cleanup plan.
- Avoid hiding warnings permanently.
- CI should match local developer expectations.

## More Coding Examples

### Example 1: Strict New Module

```swift
.target(
    name: "NewCheckoutCore",
    swiftSettings: [
        .treatAllWarnings(as: .error),
    ]
)
```

### Example 2: Allow Deprecated APIs Temporarily

```swift
.target(
    name: "UIKitBridge",
    swiftSettings: [
        .treatAllWarnings(as: .error),
        .treatWarning("DeprecatedDeclaration", as: .warning),
    ]
)
```

This keeps CI strict while allowing a known migration category.

## Common Mistakes

- Turning off warnings broadly.
- Treating all warnings as errors in unstable generated code without a plan.
- Downgrading warnings forever.
- Applying one warning policy to every target.
- Not documenting diagnostic group exceptions.

## Interview Guide

Junior:

SwiftPM can configure how warnings are treated during build.

Mid-level:

Swift 6.2 supports precise warning control by diagnostic group in `Package.swift`.

Senior:

Warning policy is part of codebase health. I use strict settings for new code, targeted exceptions for migration, and documented cleanup plans for downgraded warnings.

## Practice

1. Add `.treatAllWarnings(as: .error)` to a target.
2. Downgrade `DeprecatedDeclaration` to warning.
3. Design separate warning policies for core and legacy targets.
4. Explain why warning exceptions need owners.
