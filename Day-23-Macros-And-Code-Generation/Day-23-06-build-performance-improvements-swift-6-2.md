# Day 23: Build Performance Improvements From Swift 6.2

## What Changed In Swift 6.2

Swift 6.2 improved clean build times for projects using macro-based APIs.

Before this improvement, projects often had to fetch and build `swift-syntax` from source before building macro projects. That was expensive, especially in CI.

SwiftPM now supports prebuilt `swift-syntax` dependencies for macros, reducing that build cost.

## Why Macro Builds Were Expensive

Macro implementations depend on SwiftSyntax.

SwiftSyntax is powerful but large. Building it from source could noticeably slow:

- Clean builds
- CI builds
- Release builds
- Fresh checkouts
- Developer onboarding

## Why This Matters For iOS Teams

Macro-based libraries are common in modern Swift:

- Swift Testing
- Observation
- Dependencies/testing helpers
- Custom code generation
- Third-party macro packages

Build speed affects developer feedback loops.

## Senior Build-Performance Mindset

A senior iOS engineer asks:

- Does this macro dependency slow clean builds?
- Does CI cache it effectively?
- Is this macro used in hot targets?
- Can we reduce macro use in frequently built modules?
- Are prebuilt SwiftSyntax dependencies available?
- Does the generated code justify the build cost?

## Real iOS Use Case

If a package adds a macro for one small convenience API but increases CI time significantly, it may not be worth it.

If a macro removes thousands of lines of test/mock boilerplate and Swift 6.2 avoids the heavy SwiftSyntax build cost, it may be a better tradeoff.

## Senior iOS Engineer Artifact

```text
Artifact: Macro Build Cost Review
Macro package:
Targets using it:
Clean build impact:
Incremental build impact:
CI cache behavior:
SwiftSyntax prebuilt support:
Generated-code value:
Decision: adopt / isolate / avoid
```

## More Coding Examples

### Example 1: Isolate Macro Use

```text
App
  FeatureA
  FeatureB
  TestSupportWithMacros
```

Put macro-heavy test helpers in test/support targets when production code does not need them.

### Example 2: Check Dependency Purpose

```swift
// Before adding a macro package:
// 1. Measure clean build.
// 2. Add dependency.
// 3. Measure clean build again.
// 4. Decide based on evidence.
```

## Common Mistakes

- Adding macro packages without build measurement.
- Using macros in low-level modules imported everywhere.
- Ignoring CI cache setup.
- Keeping macro dependencies for tiny convenience.
- Not updating toolchains to benefit from Swift 6.2 improvements.

## Interview Guide

Junior:

Macros can affect build time because macro implementations depend on tooling like SwiftSyntax.

Mid-level:

Swift 6.2 improves macro build performance by supporting prebuilt SwiftSyntax dependencies.

Senior:

I treat macro adoption as an engineering tradeoff. I measure build impact, isolate macro-heavy dependencies, and verify that generated-code value justifies the cost.

## Practice

1. Explain why SwiftSyntax affected macro builds.
2. Design a build-cost review for a macro package.
3. Decide where macro-heavy test support should live.
4. Explain Swift 6.2's macro build improvement in an interview.
