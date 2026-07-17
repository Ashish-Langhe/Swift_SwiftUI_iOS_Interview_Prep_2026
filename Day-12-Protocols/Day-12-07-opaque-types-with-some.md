# Day 12: Opaque Types With Some

## Core Idea

`some Protocol` hides a concrete return type while preserving it for the compiler.

```swift
func makeTitle() -> some CustomStringConvertible {
    "Hello"
}
```

SwiftUI uses this heavily:

```swift
var body: some View {
    Text("Hello")
}
```

## Any Vs Some

`any` means any conforming type at runtime.

`some` means one specific hidden concrete type chosen by the implementation.

## Interview Levels

Junior: `some` returns a value conforming to a protocol.

Senior: `some` preserves static type information and optimization while hiding implementation details. It is great for return positions, especially SwiftUI views.

## Quick Notes

- Opaque type.
- Concrete type hidden from caller.
- Compiler still knows exact type.
- Common in SwiftUI.

## Interview Depth

Junior answer: `some Protocol` returns a value that conforms to a protocol.

Mid-level answer: The implementation chooses one concrete type, but callers do not need to know the exact type.

Senior answer: `some` hides implementation while preserving static type information and optimization. It is especially useful in return positions, such as SwiftUI `some View`.

iOS use case:

```swift
var body: some View {
    Text("Profile")
}
```

Common mistakes: thinking `some` means any type can be returned from different branches, confusing with `any`, using where stored existential is needed.

Practice: explain `some View`, compare any/some, identify branch return issue.
