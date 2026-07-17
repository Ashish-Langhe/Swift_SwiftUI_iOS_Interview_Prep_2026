# Day 3: Functions Interview Guide

## What You Will Learn

- Junior, mid-level, and senior-level function answers
- Function design principles
- Practical iOS use cases
- Common coding exercises
- Production-level reasoning around parameters, return values, closures, and side effects

## Day 3 Summary

Functions are reusable blocks of behavior. In Swift, functions are also values, which means they can be stored, passed, and returned.

Day 3 topics:

- Function declaration
- Parameters and return values
- Argument labels
- Default parameters
- Variadic parameters
- `inout`
- Function types
- Nested functions
- Higher-order functions introduction

## One-Minute Interview Answer

Functions in Swift are declared with `func` and are used to organize reusable behavior. A good function has a clear name, meaningful parameters, and an accurate return type. Swift function signatures are part of API design, so argument labels should make calls readable. I use default parameters for common safe defaults, variadic parameters for convenient small lists, and `inout` only when mutation is the clearest API. Since functions are first-class values, Swift supports callbacks, dependency injection, and higher-order functions like `map`, `filter`, and `reduce`.

## Question 1: What Is A Function?

### Junior-Level Answer

A function is a reusable block of code declared using `func`.

```swift
func greet() {
    print("Hello")
}
```

### Mid-Level Answer

Functions organize behavior, avoid duplication, and make code easier to test. They can take parameters and return values.

### Senior-Level Answer

A function is an API boundary. Its name, parameters, return type, error behavior, and side effects define a contract with callers. Good functions are focused, intention-revealing, testable, and explicit about mutation or failure.

## Question 2: What Is The Difference Between Parameter And Argument?

Answer:

A parameter is the named input in the function declaration. An argument is the actual value passed when calling the function.

```swift
func greet(name: String) {
    print(name)
}

greet(name: "Aarav")
```

`name` is the parameter. `"Aarav"` is the argument.

## Question 3: How Do Return Values Work?

Answer:

A return value is the output of a function. The return type is written after `->`.

```swift
func square(_ number: Int) -> Int {
    number * number
}
```

Senior note:

The return type should model the real outcome. Use optional when absence is valid, `throws` when failure needs explanation, `Result` when outcome must be stored or passed, and custom types when the result has domain meaning.

## Question 4: Why Are Argument Labels Important?

### Junior-Level Answer

Argument labels make function calls easier to read.

```swift
move(from: "Home", to: "Settings")
```

### Senior-Level Answer

Argument labels are API design. They reduce ambiguity, especially when multiple parameters have the same type. A function call should read naturally and reveal the role of each argument.

## Question 5: When Should You Use Default Parameters?

Answer:

Use default parameters when there is a common, safe value.

```swift
func showToast(message: String, duration: TimeInterval = 2.0) { }
```

Senior note:

Avoid defaults that hide important decisions. If many parameters have defaults, consider a configuration object.

## Question 6: What Are Variadic Parameters?

Answer:

Variadic parameters accept zero or more values.

```swift
func sum(_ numbers: Int...) -> Int {
    numbers.reduce(0, +)
}
```

Senior note:

Use variadic parameters for small manually listed values. Use arrays when values are already dynamic or collected.

## Question 7: What Is Inout?

Answer:

`inout` lets a function modify the caller's variable.

```swift
func increment(_ value: inout Int) {
    value += 1
}

var count = 0
increment(&count)
```

Senior note:

`inout` is explicit mutation across a function boundary. It should be intentional. For many business cases, returning a new value is clearer and safer.

## Question 8: What Is A Function Type?

Answer:

A function type describes parameters and return type.

```swift
let operation: (Int, Int) -> Int
```

Senior note:

Function types enable callbacks, dependency injection, strategy patterns, and test seams. Use `typealias` for complex signatures.

```swift
typealias ValidationRule = (String) -> Bool
```

## Question 9: What Is A Nested Function?

Answer:

A nested function is declared inside another function and is available only inside that outer function.

```swift
func validate(_ text: String) -> Bool {
    func isNotEmpty() -> Bool {
        !text.isEmpty
    }

    return isNotEmpty()
}
```

Senior note:

Nested functions are good for local helpers, but extract them when they need reuse, separate testing, or clearer ownership.

## Question 10: What Are Higher-Order Functions?

Answer:

Higher-order functions take functions as input or return functions.

```swift
let doubled = [1, 2, 3].map { $0 * 2 }
```

Common examples:

- `map`
- `filter`
- `reduce`
- `compactMap`
- `sorted`

Senior note:

Use higher-order functions when they express transformation clearly. Avoid using `map` for side effects and avoid overly clever chains.

## Real iOS Use Case 1: Login Validation

```swift
enum LoginValidationError: Error {
    case invalidEmail
    case weakPassword
}

func validateLogin(email: String, password: String) throws {
    guard email.contains("@") else {
        throw LoginValidationError.invalidEmail
    }

    guard password.count >= 8 else {
        throw LoginValidationError.weakPassword
    }
}
```

Discussion:

- Parameters model input.
- `throws` models failure.
- Guards keep validation readable.

## Real iOS Use Case 2: Mapping API To View Model

```swift
struct ProductResponse {
    let id: Int
    let title: String
    let price: Decimal
}

struct ProductRowViewModel {
    let title: String
    let subtitle: String
}

func makeProductRows(from responses: [ProductResponse]) -> [ProductRowViewModel] {
    responses.map { response in
        ProductRowViewModel(
            title: response.title,
            subtitle: "Price: \(response.price)"
        )
    }
}
```

Discussion:

- Function transforms data.
- Input and output are explicit.
- `map` is appropriate because every response becomes one row.

## Real iOS Use Case 3: Dependency Injection With Function Type

```swift
struct SignupViewModel {
    let isValidEmail: (String) -> Bool

    func canSubmit(email: String) -> Bool {
        isValidEmail(email)
    }
}

let viewModel = SignupViewModel { email in
    email.contains("@")
}
```

Discussion:

- The validation behavior is injected.
- Tests can pass different validation logic.
- This avoids hard-coding dependencies.

## Real iOS Use Case 4: Callback

```swift
func loadProfile(completion: @escaping (Result<String, Error>) -> Void) {
    completion(.success("Profile loaded"))
}
```

Discussion:

- Completion handler is a function type.
- `@escaping` is needed because callback may run later.
- Modern Swift often uses `async throws` instead, but callbacks are still common.

## Junior-Level Rapid Revision

- Functions use `func`.
- Parameters are inputs.
- Return values are outputs.
- Argument labels appear when calling functions.
- Default parameters provide fallback values.
- Variadic parameters accept many values.
- `inout` modifies the original variable.

## Mid-Level Rapid Revision

- Good function signatures improve readability.
- Return optionals when absence is valid.
- Use throwing functions when failure needs detail.
- Use default parameters for common safe behavior.
- Use `typealias` for complex function types.
- Use nested functions for local helpers.
- Use `map`, `filter`, and `reduce` for clear transformations.

## Senior-Level Rapid Revision

- Function signatures are API contracts.
- Argument labels are part of Swift API design.
- Mutating external state should be explicit.
- Prefer pure functions for business logic where possible.
- Avoid hidden side effects in helper functions.
- Choose between optional, throwing, `Result`, and domain types intentionally.
- Stored closures require attention to escaping behavior and captures.
- Higher-order functions should improve clarity, not just reduce line count.

## Common Mistakes

### Too Many Parameters

```swift
func createUser(name: String, email: String, city: String, age: Int, role: String) { }
```

Consider a request model:

```swift
struct CreateUserRequest {
    let name: String
    let email: String
    let city: String
    let age: Int
    let role: String
}
```

### Using Inout For Everything

Avoid mutation-heavy APIs when a return value is clearer.

```swift
func normalized(_ text: String) -> String {
    text.trimmingCharacters(in: .whitespaces)
}
```

### Overusing Higher-Order Chains

Readable:

```swift
let names = users
    .filter { $0.isActive }
    .map { $0.name }
```

Too much logic inside closures should be extracted.

## Coding Exercise 1: Price Calculator

```swift
func totalPrice(quantity: Int, unitPrice: Decimal, discount: Decimal = 0) -> Decimal {
    let subtotal = Decimal(quantity) * unitPrice
    return subtotal - discount
}
```

## Coding Exercise 2: Variadic Validation

```swift
func allNonEmpty(_ values: String...) -> Bool {
    values.allSatisfy { !$0.isEmpty }
}
```

## Coding Exercise 3: Function Type

```swift
func apply(_ operation: (Int, Int) -> Int, to first: Int, and second: Int) -> Int {
    operation(first, second)
}

let result = apply(+, to: 10, and: 20)
```

## Coding Exercise 4: Higher-Order Function

```swift
func activeUserNames(from users: [User]) -> [String] {
    users
        .filter { $0.isActive }
        .map { $0.name }
}
```

## Final Interview Questions

1. What is a function?
2. What is the difference between parameter and argument?
3. Why are argument labels important in Swift?
4. When should you use default parameters?
5. What is a variadic parameter?
6. What does `inout` do?
7. Why should `inout` be used carefully?
8. What is a function type?
9. What is a nested function?
10. What is a higher-order function?
11. When should you use `map`, `filter`, and `reduce`?
12. How do you design a good function signature?
13. What makes a function easy to test?
14. How do callbacks relate to function types?
15. How would you refactor a function with too many parameters?

## Deep Dive Expansion

### Why This Topic Matters In Real Apps

Functions Interview Guide is not only a syntax topic. In production Swift, it affects clear API design, parameter labels, ownership of side effects, and composability. A junior developer should be able to recognize the syntax and explain the basic behavior. A mid-level developer should know when the feature improves clarity or safety. A senior developer should understand how it changes API design, testability, performance, or long-term maintenance.

In iOS work, this usually appears while building service methods, validation helpers, and reusable transformation functions. The important question is not "Can I write the syntax?" The important question is "Does this code make invalid states harder, make ownership clearer, and make future changes safer?"

### Beginner To Senior Progression

Beginner level:

- Define the concept in plain language.
- Write the smallest working example.
- Recognize the compiler error when the feature is used incorrectly.
- Explain what happens at the call site.

Mid-level level:

- Choose this feature over a nearby alternative for a clear reason.
- Handle edge cases, nil/error/cancellation/performance concerns where relevant.
- Keep the code readable for the next developer.
- Write tests around the behavior, not just the implementation detail.

Senior level:

- Explain the design tradeoff and the failure mode it prevents.
- Understand how this feature behaves across module, actor, memory, or API boundaries.
- Design examples that scale from a small screen to a larger feature.
- Avoid exposing implementation details as permanent API.

### Production-Style Example Pattern

Use this checklist when applying Functions Interview Guide in an app feature:

1. Identify the owner of the data or behavior.
2. Decide whether the value should be mutable, immutable, optional, throwing, async, isolated, or private.
3. Keep the public surface small and intention-revealing.
4. Add one realistic failure path, not only the happy path.
5. Check whether the code is still understandable from the call site.

```swift
struct FeatureState: Equatable {
    var isLoading: Bool
    var message: String?
    var canRetry: Bool
}

func makeInitialState() -> FeatureState {
    FeatureState(isLoading: false, message: nil, canRetry: false)
}
```

This small pattern is intentionally simple: define the state, control mutation through a narrow function, and make the result easy to inspect in tests.

### Edge Cases To Think About

- What happens when the input is empty, nil, duplicated, delayed, or invalid?
- What happens when this code is called repeatedly from a scrolling list or fast-changing UI?
- Does this API expose too much mutable state?
- Does the implementation assume a specific ordering, lifetime, actor, or thread?
- Will this still be easy to test after the feature grows?

### Topic-Specific Senior Notes

- Turn each answer into a story: situation, decision, tradeoff, result, and verification.
- For senior interviews, avoid only definitions. Explain why the design prevents bugs in a real app.

### Common Interview Follow-Ups

Be ready for these follow-ups:

- Why did you choose this approach instead of the simpler alternative?
- What bug could happen if this is implemented carelessly?
- How would this behave in a large codebase with multiple modules?
- How would you test this without relying on UI screenshots?
- What changes when this code becomes async, public, generic, or shared?

### Strong Interview Framing

A strong answer should sound like this:

```text
I understand the basic syntax, but I also think about ownership and boundaries. I choose the approach that keeps state valid, makes the call site clear, and avoids unnecessary coupling. In a production iOS app, I would also consider testing, cancellation or error behavior where relevant, and whether this should remain an implementation detail or become part of a public API.
```

### Extra Practice

1. Write a minimal example that shows Functions Interview Guide correctly.
2. Write a second example that demonstrates a common mistake.
3. Refactor the mistake into a safer version.
4. Explain the difference between the beginner solution and the production-ready solution.
5. Turn one answer into a real project story with tradeoffs and verification.

## Applied Day-Level Example

The following example is not the only way to use Functions Interview Guide, but it shows the kind of production shape you should connect this topic to:

```swift
struct PasswordRule {
    let message: String
    let validate: (String) -> Bool
}

func validatePassword(_ password: String, rules: [PasswordRule]) -> [String] {
    rules.compactMap { rule in
        rule.validate(password) ? nil : rule.message
    }
}

let rules = [
    PasswordRule(message: "Use at least 8 characters") { $0.count >= 8 },
    PasswordRule(message: "Add one number") { $0.contains(where: \.isNumber) }
]
```

This connects function design to reuse. The validation function has clear inputs and output, while each rule is injectable and easy to test.

### How To Study This Example

- Identify which part is syntax and which part is design.
- Ask what invalid state the example prevents.
- Ask what would need to change if the feature became async, generic, public, or shared across modules.
- Rewrite the example with one deliberate bug, then explain how a reviewer should catch it.
- Turn the example into a two-minute interview explanation with tradeoffs.

