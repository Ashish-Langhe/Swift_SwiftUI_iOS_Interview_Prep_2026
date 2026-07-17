# Day 4: Optional Pattern Matching

## Core Idea

Because optional is conceptually an enum, you can pattern match it.

```swift
let name: String? = "Aarav"

switch name {
case .some(let value):
    print(value)
case .none:
    print("No name")
}
```

Most daily code uses `if let`, but optional pattern matching is useful in advanced control flow.

## If Case With Optionals

```swift
let score: Int? = 90

if case .some(let value) = score {
    print(value)
}
```

Equivalent common style:

```swift
if let score {
    print(score)
}
```

## Optional Pattern Shorthand

Swift supports a pattern shorthand:

```swift
let score: Int? = 90

if case let value? = score {
    print(value)
}
```

`value?` means match `.some(value)`.

## For Case With Optionals

```swift
let values: [Int?] = [1, nil, 3, nil, 5]

for case let value? in values {
    print(value)
}
```

This iterates only non-nil values.

Often, `compactMap` is clearer:

```swift
for value in values.compactMap({ $0 }) {
    print(value)
}
```

## Switch With Optional And Conditions

```swift
let age: Int? = 20

switch age {
case .some(let value) where value >= 18:
    print("Adult")
case .some:
    print("Minor")
case .none:
    print("Unknown age")
}
```

## Real iOS Use Cases

### Optional View State Data

```swift
let selectedUserId: String? = "user-101"

switch selectedUserId {
case .some(let id):
    print("Load user \(id)")
case .none:
    print("Show empty selection")
}
```

### Filtering Optional API Values

```swift
let rawPrices: [Decimal?] = [100, nil, 250]

for case let price? in rawPrices {
    print("Price: \(price)")
}
```

### Combining Enum And Optional

```swift
enum ScreenState {
    case loaded(userId: String?)
    case loading
}

let state = ScreenState.loaded(userId: "abc")

switch state {
case .loaded(let userId?):
    print("Loaded user \(userId)")
case .loaded(nil):
    print("Loaded without user")
case .loading:
    print("Loading")
}
```

## Junior-Level Interview Answer

Optional pattern matching means checking whether an optional is `.some` or `.none` using `switch` or pattern syntax.

## Mid-Level Interview Answer

Optional pattern matching is useful when optionals are part of a larger pattern, such as switching with conditions or iterating through arrays of optional values.

## Senior-Level Interview Answer

Optional pattern matching is powerful but should be used for clarity, not cleverness. In normal code, `if let` is usually clearer. I use optional patterns when they compose naturally with enum states, tuple matching, `where` clauses, or `for case` filtering.

## Quick Interview Notes

- Optional is pattern matchable as `.some` and `.none`.
- `if let` is usually simpler for basic unwrapping.
- `for case let value?` iterates non-nil optional values.
- `where` can add conditions to optional patterns.
- Optional pattern matching is useful with enums and complex state.

## Practice Questions

1. How can you switch over an optional?
2. What does `.some` mean?
3. What does `.none` mean?
4. What does `for case let value?` do?
5. When is optional pattern matching better than `if let`?

