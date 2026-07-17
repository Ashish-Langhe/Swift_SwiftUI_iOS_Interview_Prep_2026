# Day 3: Higher-Order Functions Introduction

## Core Idea

A higher-order function is a function that takes another function as input, returns a function, or both.

Swift collections provide common higher-order functions:

- `map`
- `filter`
- `reduce`
- `compactMap`
- `flatMap`
- `forEach`
- `sorted`

## Map

`map` transforms each element.

```swift
let numbers = [1, 2, 3]
let doubled = numbers.map { $0 * 2 }
```

Result:

```swift
[2, 4, 6]
```

## Filter

`filter` keeps elements that match a condition.

```swift
let numbers = [1, 2, 3, 4, 5]
let evenNumbers = numbers.filter { $0.isMultiple(of: 2) }
```

Result:

```swift
[2, 4]
```

## Reduce

`reduce` combines values into one result.

```swift
let prices = [100, 200, 300]
let total = prices.reduce(0) { partialResult, price in
    partialResult + price
}
```

Short form:

```swift
let total = prices.reduce(0, +)
```

## CompactMap

`compactMap` transforms values and removes nil results.

```swift
let strings = ["1", "two", "3"]
let numbers = strings.compactMap { Int($0) }
```

Result:

```swift
[1, 3]
```

## Sorted

```swift
let names = ["Meera", "Aarav", "Kabir"]
let sortedNames = names.sorted { $0 < $1 }
```

Short form:

```swift
let sortedNames = names.sorted()
```

## ForEach

```swift
let topics = ["Variables", "Functions", "Optionals"]

topics.forEach { topic in
    print(topic)
}
```

Use `forEach` for simple side effects. Use a normal `for` loop when you need `break`, `continue`, or early return from the outer scope.

## Real iOS Use Cases

### Mapping API Models To View Models

```swift
struct UserResponse {
    let id: Int
    let name: String
}

struct UserRowViewModel {
    let title: String
}

let responses = [
    UserResponse(id: 1, name: "Aarav"),
    UserResponse(id: 2, name: "Meera")
]

let rows = responses.map { response in
    UserRowViewModel(title: response.name)
}
```

### Filtering Search Results

```swift
let users = ["Aarav", "Meera", "Kabir"]
let query = "aa"

let results = users.filter {
    $0.localizedCaseInsensitiveContains(query)
}
```

### Calculating Total

```swift
let amounts: [Decimal] = [100, 250, 50]
let total = amounts.reduce(0, +)
```

### Parsing IDs

```swift
let rawIds = ["101", "abc", "202"]
let ids = rawIds.compactMap { Int($0) }
```

## Higher-Order Functions Vs Loops

Loop:

```swift
var names: [String] = []

for user in responses {
    names.append(user.name)
}
```

Higher-order function:

```swift
let names = responses.map { $0.name }
```

Use the style that is clearest. Higher-order functions are not automatically better.

## Common Mistakes

### Using Map For Side Effects

Avoid:

```swift
users.map { print($0) }
```

Prefer:

```swift
users.forEach { print($0) }
```

Or:

```swift
for user in users {
    print(user)
}
```

### Making Chains Too Clever

```swift
let result = users
    .filter { $0.isActive }
    .map { $0.name }
    .sorted()
```

This is fine. But if each closure becomes complex, extract named helpers.

## Junior-Level Interview Answer

A higher-order function is a function that takes another function or closure as a parameter, or returns one. Examples are `map`, `filter`, and `reduce`.

## Mid-Level Interview Answer

Higher-order functions make collection transformations concise. `map` transforms, `filter` selects, `reduce` combines, and `compactMap` transforms while removing nil.

## Senior-Level Interview Answer

Higher-order functions are tools for expressing data transformations declaratively. I use them when they improve clarity and keep transformations local. I avoid long chains with complex closures, avoid `map` for side effects, and consider performance when chaining over large collections. In UI code, mapping API models into view models is one of the most common and clean use cases.

## Quick Interview Notes

- `map` transforms.
- `filter` keeps matching values.
- `reduce` combines into one result.
- `compactMap` removes nil after transformation.
- `forEach` performs side effects.
- Use loops when control flow needs `break` or `continue`.

## Practice Questions

1. What is a higher-order function?
2. What is the difference between `map` and `filter`?
3. What does `reduce` do?
4. When should you use `compactMap`?
5. Why should you avoid `map` for side effects?

