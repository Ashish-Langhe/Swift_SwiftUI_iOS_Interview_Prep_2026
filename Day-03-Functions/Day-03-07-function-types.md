# Day 3: Function Types

## Core Idea

Functions have types. A function type describes its parameter types and return type.

```swift
func add(_ a: Int, _ b: Int) -> Int {
    a + b
}

let operation: (Int, Int) -> Int = add
let result = operation(2, 3)
```

The type `(Int, Int) -> Int` means:

- Takes two `Int` values
- Returns an `Int`

## Function As A Value

You can store functions in variables or constants.

```swift
func multiply(_ a: Int, _ b: Int) -> Int {
    a * b
}

let mathOperation = multiply
print(mathOperation(4, 5))
```

## Function As A Parameter

You can pass a function into another function.

```swift
func perform(_ operation: (Int, Int) -> Int, a: Int, b: Int) -> Int {
    operation(a, b)
}

let result = perform(add, a: 10, b: 20)
```

This is a foundation for callbacks and higher-order functions.

## Function As A Return Value

```swift
func makeMultiplier(by factor: Int) -> (Int) -> Int {
    func multiplier(value: Int) -> Int {
        value * factor
    }

    return multiplier
}

let double = makeMultiplier(by: 2)
print(double(10))
```

## Optional Function Types

```swift
var completion: (() -> Void)?

completion = {
    print("Completed")
}

completion?()
```

This is common for callbacks.

## Typealias For Function Types

Function types can become hard to read. Use `typealias`.

```swift
typealias CompletionHandler = (Result<Data, Error>) -> Void

func loadData(completion: CompletionHandler) {
    completion(.success(Data()))
}
```

## Real iOS Use Cases

### Completion Handler

```swift
func fetchUser(completion: @escaping (String) -> Void) {
    completion("Aarav")
}
```

`@escaping` is needed when the closure may be called after the function returns.

### Button Action Concept

```swift
let onTap: () -> Void = {
    print("Button tapped")
}

onTap()
```

### Dependency Injection

```swift
struct LoginValidator {
    let isValidEmail: (String) -> Bool
}

let validator = LoginValidator { email in
    email.contains("@")
}
```

## Junior-Level Interview Answer

A function type defines what parameters a function accepts and what it returns.

```swift
(Int, Int) -> Int
```

## Mid-Level Interview Answer

Functions are first-class values in Swift. We can store them in variables, pass them as parameters, and return them from other functions. This enables callbacks, dependency injection, and flexible behavior.

## Senior-Level Interview Answer

Function types let behavior become data. This is powerful for composition, dependency injection, testability, and asynchronous APIs. I use type aliases when function signatures become complex. I also pay attention to escaping closures, capture lists, actor isolation, and retain cycles when function values are stored or called later.

## Quick Interview Notes

- Function types use `(Parameters) -> Return`.
- Functions can be stored in variables.
- Functions can be passed as arguments.
- Functions can be returned.
- Use `typealias` for readability.
- Stored callbacks often involve escaping closures.

## Practice Questions

1. What is the type of a function that takes `String` and returns `Bool`?
2. Can functions be stored in variables?
3. Why use a function as a parameter?
4. What is a callback?
5. When is `typealias` useful?

