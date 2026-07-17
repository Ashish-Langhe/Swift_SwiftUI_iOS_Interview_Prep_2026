# Day 23: Macros And Code Generation Interview Guide

## One-Minute Senior Answer

Swift macros generate code at compile time. Freestanding macros use `#`, attached macros use `@`, and macro implementations rely on SwiftSyntax to inspect and produce Swift syntax. A senior iOS engineer uses macros when they meaningfully reduce boilerplate, improve compile-time validation, or produce better diagnostics. I avoid macros when a function, protocol extension, property wrapper, or simple script is clearer. I also review generated API, diagnostics, test coverage, and build performance.

## Junior Questions

What is a macro?

A compile-time code generation feature.

What is a freestanding macro?

A macro used with `#`, such as `#expect`.

What is an attached macro?

A macro attached to a declaration with `@`, such as `@Test`.

## Mid-Level Questions

Why use macros?

To reduce repetitive code, validate patterns at compile time, and improve source-aware diagnostics.

What is SwiftSyntax?

A library for inspecting and generating Swift syntax trees. Macro implementations use it.

What is a good macro use case?

Testing assertions, mock generation, localization validation, feature flags, or generated endpoint declarations.

## Senior Questions

How do you decide whether to use a macro?

I compare it against simpler Swift alternatives, estimate build/debugging cost, define generated code shape, and require clear diagnostics and tests.

What are macro risks?

Hidden behavior, poor diagnostics, generated API instability, build-time cost, and difficult debugging.

What changed in Swift 6.2?

Swift 6.2 improved macro build performance by supporting prebuilt `swift-syntax` dependencies, reducing clean build cost for macro-heavy projects.

## Senior iOS Engineer Artifact

```text
Artifact: Macro Adoption Review
Problem:
Manual boilerplate:
Macro type: freestanding / attached
Generated output:
Diagnostics:
Test strategy:
Build cost:
SwiftSyntax dependency impact:
Alternative:
Decision:
```

## Real iOS Scenario

You want to generate mocks for service protocols.

Senior answer:

```text
I would first check whether manual stubs or a test helper are enough. If the project has many service protocols and tests are slowed by boilerplate, a @Mockable macro may be justified. I would require expansion tests for async/throws methods, clear diagnostics for unsupported protocol shapes, and a build-performance check in CI.
```

## Common Traps

- Macro because it feels modern.
- No tests for macro expansion.
- Bad diagnostics.
- Generated public API without review.
- Build cost ignored.
- Business logic hidden in generated code.

## More Coding Examples

### Example 1: Freestanding Testing Macro

```swift
#expect(viewModel.state == .loaded)
```

### Example 2: Attached Test Macro

```swift
@Test
func loading_setsLoadedState() async throws {
    #expect(true)
}
```

### Example 3: Conceptual Mock Macro

```swift
@Mockable
protocol UserLoading {
    func user(id: String) async throws -> User
}
```

## Interview Closing Answer

Macros are powerful when they make correctness cheaper. At senior level, I focus on whether the macro improves the developer experience while keeping generated code transparent, diagnostics useful, and builds healthy.

## Practice Prompts

1. Compare a function, property wrapper, and macro for one use case.
2. Design diagnostics for a localization macro.
3. Sketch a `@Mockable` expansion.
4. Explain SwiftSyntax to a junior engineer.
5. Explain Swift 6.2 macro build performance improvements.
