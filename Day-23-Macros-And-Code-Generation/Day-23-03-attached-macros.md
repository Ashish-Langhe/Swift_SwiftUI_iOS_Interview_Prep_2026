# Day 23: Attached Macros

## What Attached Macros Are

Attached macros attach to declarations using `@`.

Examples:

```swift
@Test
func total_isCalculatedCorrectly() { }

@Observable
final class ProfileModel { }
```

Attached macros can add code around or inside declarations depending on macro role.

## Common Attached Macro Roles

Attached macros can be:

- Peer macros
- Member macros
- Member attribute macros
- Accessor macros
- Conformance macros
- Extension macros

Each role describes what code the macro can add.

## Real iOS Example: Observable

```swift
@Observable
final class CartModel {
    var items: [CartItem] = []
}
```

The macro expands into observation-related code.

This removes boilerplate while keeping the model declarative.

## Conceptual Example: Auto-Mock

```swift
@Mockable
protocol UserLoading {
    func user(id: String) async throws -> User
}
```

A macro could generate:

- `MockUserLoading`
- call recording
- configurable return values
- error throwing behavior

Senior caution:

Mock generation is useful only if generated mocks remain readable and test failures stay clear.

## Attached Macro Declaration

```swift
@attached(member, names: named(mock))
public macro Mockable() =
    #externalMacro(module: "MockMacros", type: "MockableMacro")
```

The declaration describes what the macro promises to generate.

## Senior iOS Engineer Artifact

```text
Artifact: Attached Macro Expansion Review
Macro:
Attached to:
Generated members:
Generated conformances:
Access control:
Public API impact:
Source compatibility:
Debuggability:
```

Senior lens:

- Attached macros change the shape of a type.
- Generated members can become API.
- Access control must be intentional.
- Macro expansion should be part of review.

## More Coding Examples

### Example 1: Test Declaration

```swift
@Test
func login_withValidCredentials_succeeds() async throws {
    #expect(true)
}
```

`@Test` marks the function as a Swift Testing test.

### Example 2: Debug Description

```swift
@DebugDescription
struct User: CustomDebugStringConvertible {
    let id: String
    let name: String

    var debugDescription: String {
        "#\\(id) \\(name)"
    }
}
```

Macros can improve tooling and debugging, not only app runtime code.

## Common Mistakes

- Forgetting generated members may affect API surface.
- Using macros that hide too much behavior.
- Not testing macro expansion.
- Allowing generated names to collide.
- Ignoring access control of generated code.

## Interview Guide

Junior:

Attached macros use `@` and attach to declarations.

Mid-level:

They can generate members, conformances, accessors, or peer declarations.

Senior:

Attached macros are type-shaping tools. I review generated API, access control, source compatibility, diagnostics, and testability.

## Practice

1. Explain `@Test` as an attached macro.
2. Design a `@Mockable` macro.
3. List generated members it should create.
4. Explain why access control matters for generated members.
