# Day 3: Parameters And Return Values

## Core Idea

Parameters are inputs to a function. Return values are outputs from a function.

```swift
func greet(name: String) -> String {
    "Hello, \(name)"
}

let message = greet(name: "Aarav")
```

Here:

- `name` is a parameter
- `"Aarav"` is an argument
- `String` after `->` is the return type
- `message` receives the returned value

## Function With Parameters

```swift
func printUser(name: String, age: Int) {
    print("\(name) is \(age) years old")
}

printUser(name: "Meera", age: 28)
```

Parameters let the same function work with different values.

## Function With Return Value

```swift
func add(_ first: Int, _ second: Int) -> Int {
    first + second
}

let result = add(10, 20)
```

The return type must match the returned value.

## Multiple Return Paths

```swift
func passStatus(score: Int) -> String {
    if score >= 50 {
        return "Pass"
    } else {
        return "Fail"
    }
}
```

Every possible path must return a `String`.

## Returning Optionals

Use optional returns when a function may not produce a value.

```swift
func firstCharacter(in text: String) -> Character? {
    text.first
}

let character = firstCharacter(in: "Swift")
```

This is better than returning a fake placeholder.

## Returning Tuples

Use tuples for small, related return values.

```swift
func minMax(numbers: [Int]) -> (min: Int, max: Int)? {
    guard let first = numbers.first else {
        return nil
    }

    var minValue = first
    var maxValue = first

    for number in numbers {
        if number < minValue { minValue = number }
        if number > maxValue { maxValue = number }
    }

    return (minValue, maxValue)
}
```

Use a struct when the return value is part of your domain and will grow over time.

## Returning Result

For success/failure APIs, `Result` can be useful.

```swift
enum LoginError: Error {
    case invalidEmail
}

func validate(email: String) -> Result<String, LoginError> {
    guard email.contains("@") else {
        return .failure(.invalidEmail)
    }

    return .success(email)
}
```

Throwing functions are often more idiomatic for error propagation, but `Result` is useful for storing or passing outcomes.

## Real iOS Use Cases

### Validation

```swift
func isValidEmail(_ email: String) -> Bool {
    email.contains("@") && email.contains(".")
}
```

### Formatting Currency

```swift
func formattedPrice(amount: Decimal, currencyCode: String) -> String {
    "\(currencyCode) \(amount)"
}
```

### Building URL

```swift
func userURL(baseURL: URL, userId: String) -> URL {
    baseURL.appendingPathComponent("users").appendingPathComponent(userId)
}
```

## Junior-Level Interview Answer

Parameters are values passed into a function. Return values are values sent back from a function.

```swift
func double(_ number: Int) -> Int {
    number * 2
}
```

## Mid-Level Interview Answer

Parameters define what a function needs to do its job. Return values define what the caller receives. I use optional return types when a value may not exist, tuples for small grouped results, and custom structs for meaningful domain results.

## Senior-Level Interview Answer

Function signatures are API design. The parameter list should reveal dependencies and the return type should model the outcome accurately. Returning `nil`, throwing an error, returning `Result`, or returning a domain object are different design choices. I choose based on whether failure is expected, recoverable, and meaningful to the caller.

## Quick Interview Notes

- Parameters are inputs.
- Arguments are actual values passed during the call.
- Return values are outputs.
- `-> Type` declares the return type.
- Optional returns represent possible absence.
- Tuples are fine for small related values.
- Domain-heavy returns should usually be structs or enums.

## Practice Questions

1. What is the difference between a parameter and an argument?
2. How do you declare a return type?
3. When should a function return an optional?
4. When is a tuple return useful?
5. What makes a function signature well-designed?

