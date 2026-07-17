# Day 3: Function Declaration

## What You Will Learn

- What a function is
- How to declare and call functions
- How function names communicate intent
- How functions improve reuse, readability, and testability
- Junior to senior-level interview explanations

## Core Idea

A function is a named block of code that performs a task.

```swift
func greet() {
    print("Hello, Swift")
}

greet()
```

Function syntax:

```swift
func functionName() {
    // Code
}
```

## Why Functions Matter

Functions help you:

- Avoid duplicate code
- Give a name to a behavior
- Split large logic into smaller pieces
- Test behavior independently
- Improve readability

Without a function:

```swift
let price = 100.0
let tax = price * 0.18
let finalPrice = price + tax
```

With a function:

```swift
func finalPrice(for price: Double) -> Double {
    let tax = price * 0.18
    return price + tax
}
```

## Calling A Function

```swift
func showWelcomeMessage() {
    print("Welcome back")
}

showWelcomeMessage()
```

The code inside the function runs only when the function is called.

## Function Naming

Swift function names should read naturally at the call site.

Good:

```swift
func loadUserProfile() { }
func validateEmail() { }
func calculateTotalPrice() { }
```

Weak:

```swift
func doThing() { }
func process() { }
func data() { }
```

Clear names are especially important in interviews and production code.

## Functions Inside Types

Functions inside structs, classes, actors, and enums are usually called methods.

```swift
struct Cart {
    var items: [String]

    func itemCount() -> Int {
        items.count
    }
}

let cart = Cart(items: ["Book", "Pen"])
print(cart.itemCount())
```

## Void Functions

A function that does not return a value returns `Void`.

```swift
func logOut() {
    print("User logged out")
}
```

This is equivalent to:

```swift
func logOut() -> Void {
    print("User logged out")
}
```

Most developers omit `-> Void`.

## Single-Expression Functions

When a function has one expression, Swift can infer the return.

```swift
func square(_ number: Int) -> Int {
    number * number
}
```

This is valid because `number * number` is the only expression.

For beginners, writing `return` is sometimes clearer:

```swift
func square(_ number: Int) -> Int {
    return number * number
}
```

## Real iOS Use Cases

### ViewModel Action

```swift
final class LoginViewModel {
    func loginButtonTapped() {
        print("Validate input and start login")
    }
}
```

### Formatting Display Text

```swift
func displayName(firstName: String, lastName: String) -> String {
    "\(firstName) \(lastName)"
}
```

### Analytics Event

```swift
func trackScreenView(name: String) {
    print("Screen viewed: \(name)")
}
```

## Junior-Level Interview Answer

Question: What is a function in Swift?

Answer:

A function is a reusable block of code. We declare it using the `func` keyword and call it by using its name.

## Mid-Level Interview Answer

Question: Why do we use functions?

Answer:

Functions reduce duplication, make code more readable, and separate responsibilities. They also make logic easier to test because a function can take input and return output.

## Senior-Level Interview Answer

Question: What makes a function well-designed?

Answer:

A well-designed function has a clear purpose, a meaningful name, minimal side effects, focused inputs, and predictable output. In production Swift, I try to keep functions small enough to reason about, but not so fragmented that the flow becomes hard to follow. I also separate pure calculation functions from functions that perform side effects like network calls, navigation, persistence, or analytics.

## Quick Interview Notes

- Functions are declared with `func`.
- A function runs only when called.
- Functions improve reuse and testability.
- Functions inside types are called methods.
- `Void` means no return value.
- Single-expression functions can omit `return`.

## Points To Remember

- Name functions by what they do.
- Keep functions focused.
- Avoid hidden side effects in utility functions.
- Prefer functions that are easy to test.
- In iOS code, functions often represent user actions, formatting, validation, networking, and state updates.

## Practice Questions

1. What keyword declares a function?
2. What is the difference between declaring and calling a function?
3. What does `Void` mean?
4. What is a method?
5. What makes a function easy to test?

