# Day 23: SwiftSyntax Basics

## What SwiftSyntax Is

SwiftSyntax is a library for parsing, inspecting, and generating Swift source syntax.

Macro implementations use SwiftSyntax to understand and produce Swift code.

Key idea:

SwiftSyntax works with syntax trees, not runtime values.

## Syntax Tree Mental Model

Source code:

```swift
let name = "Ashish"
```

Can be represented as syntax nodes:

```text
Variable declaration
  binding
    identifier: name
    initializer: "Ashish"
```

Macros inspect these nodes.

## Why Macro Authors Need It

A macro implementation needs to:

- Inspect macro arguments
- Validate declaration shape
- Generate Swift syntax
- Produce diagnostics
- Preserve source locations

## Conceptual Macro Implementation

```swift
import SwiftSyntax
import SwiftSyntaxMacros

public struct ExampleMacro: ExpressionMacro {
    public static func expansion(
        of node: some FreestandingMacroExpansionSyntax,
        in context: some MacroExpansionContext
    ) throws -> ExprSyntax {
        "\"generated\""
    }
}
```

This is intentionally simple: the macro returns an expression syntax node.

## Diagnostics

Good macros produce useful diagnostics.

```text
@Mockable can only be applied to protocols.
```

Bad diagnostic:

```text
Expansion failed.
```

Senior engineers treat diagnostics as part of API quality.

## Real iOS Use Case

Auto-generating mocks:

The macro must inspect:

- Protocol name
- Function requirements
- Async/throws markers
- Return types
- Access levels
- Associated types

Then generate mock code that compiles.

## Senior iOS Engineer Artifact

```text
Artifact: SwiftSyntax Macro Implementation Plan
Input syntax:
Supported declaration shapes:
Unsupported cases:
Generated syntax:
Diagnostics:
Formatting expectations:
Tests with macro expansion:
SwiftSyntax version/build impact:
```

Senior lens:

- SwiftSyntax APIs can be verbose.
- Macro tests should verify expansion output.
- Unsupported cases need clear diagnostics.
- Generated code should be stable across formatting.

## More Coding Examples

### Example 1: Read Macro Argument Conceptually

```swift
#localized("settings.title")
```

The implementation inspects the macro argument syntax and verifies it is a string literal.

### Example 2: Generate Expression

```swift
return "String(localized: \"settings.title\")"
```

In real macro code this is built as `ExprSyntax`.

## Common Mistakes

- Treating syntax as runtime data.
- Producing vague diagnostics.
- Supporting too many cases too early.
- Not testing expansion output.
- Ignoring SwiftSyntax version/build impact.

## Interview Guide

Junior:

SwiftSyntax lets tools inspect and generate Swift source code.

Mid-level:

Macro implementations use SwiftSyntax to read syntax nodes and produce new syntax.

Senior:

SwiftSyntax is infrastructure. I keep macro scope small, test expansions, design diagnostics carefully, and consider build performance.

## Practice

1. Draw the syntax tree for a simple variable declaration.
2. Design diagnostics for a bad macro usage.
3. Explain syntax vs runtime value.
4. Write test cases for macro expansion.
