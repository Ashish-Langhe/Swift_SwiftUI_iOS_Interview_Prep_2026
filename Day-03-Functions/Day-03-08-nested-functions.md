# Day 3: Nested Functions

## Core Idea

A nested function is a function declared inside another function.

```swift
func makeGreeting(name: String) -> String {
    func prefix() -> String {
        "Hello"
    }

    return "\(prefix()), \(name)"
}
```

The nested function is available only inside the outer function.

## Why Use Nested Functions?

Use nested functions when helper logic is needed only inside one function.

```swift
func validatePassword(_ password: String) -> Bool {
    func hasMinimumLength() -> Bool {
        password.count >= 8
    }

    func hasNumber() -> Bool {
        password.contains { $0.isNumber }
    }

    return hasMinimumLength() && hasNumber()
}
```

The helpers are not visible outside `validatePassword`.

## Capturing Values

Nested functions can use values from the outer function.

```swift
func makeCounter(start: Int) -> () -> Int {
    var count = start

    func next() -> Int {
        count += 1
        return count
    }

    return next
}

let counter = makeCounter(start: 0)
print(counter())
print(counter())
```

The nested function captures `count`.

## Real iOS Use Cases

### Local Validation Helpers

```swift
func canSubmit(email: String, password: String) -> Bool {
    func isEmailValid() -> Bool {
        email.contains("@")
    }

    func isPasswordValid() -> Bool {
        password.count >= 8
    }

    return isEmailValid() && isPasswordValid()
}
```

### Local Formatting Helper

```swift
func profileSummary(name: String, city: String?) -> String {
    func cityText() -> String {
        city ?? "Unknown city"
    }

    return "\(name), \(cityText())"
}
```

### Recursive Helper

```swift
func countdown(from number: Int) {
    func printNumber(_ value: Int) {
        guard value > 0 else { return }
        print(value)
        printNumber(value - 1)
    }

    printNumber(number)
}
```

## When Not To Use Nested Functions

Avoid nested functions when the helper:

- Is useful elsewhere
- Needs independent tests
- Makes the outer function too long
- Captures too much state

In those cases, extract a private method or standalone function.

## Junior-Level Interview Answer

A nested function is a function written inside another function. It can be used only inside the outer function.

## Mid-Level Interview Answer

Nested functions are useful for local helper logic. They keep implementation details hidden and prevent helper functions from polluting the wider type or file.

## Senior-Level Interview Answer

Nested functions are a scoping tool. They are useful when helper behavior belongs exclusively to one algorithm and benefits from capturing local values. I avoid them when they reduce testability, hide too much complexity, or capture mutable state in a way that makes behavior harder to reason about.

## Quick Interview Notes

- Nested functions are declared inside functions.
- They are scoped to the outer function.
- They can capture outer values.
- They are useful for local helper logic.
- Extract them if reuse or testing becomes important.

## Practice Questions

1. What is a nested function?
2. Can a nested function access outer function values?
3. Why use nested functions?
4. When should you avoid nested functions?
5. How are nested functions related to closures?

