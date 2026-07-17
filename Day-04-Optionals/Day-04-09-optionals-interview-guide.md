# Day 4: Optionals Interview Guide

## What You Will Learn

- Junior, mid-level, and senior-level optional answers
- How to explain optional safety clearly
- How to discuss force unwraps, IUOs, nil-coalescing, and optional chaining
- Real iOS examples with API responses and view state
- Common coding exercises and final revision questions

## Day 4 Summary

Optionals are one of Swift's most important safety features. They model values that may be missing.

Day 4 topics:

- Optional meaning and memory model intuition
- Optional binding
- Optional chaining
- Nil-coalescing
- Force unwrap risks
- Implicitly unwrapped optionals
- Optional pattern matching
- Real iOS examples with API responses and view state

## One-Minute Interview Answer

An optional in Swift represents either a value or nil. It is conceptually an enum with `.some(value)` and `.none`. Swift makes absence visible in the type system, so we must handle it safely using optional binding, `guard let`, optional chaining, nil-coalescing, or pattern matching. I avoid force unwraps for dynamic data because nil would crash the app. In iOS, optionals are common for API fields, selected state, delegates, and user input, but too many optionals in a model may indicate that an enum state would be clearer.

## Question 1: What Is An Optional?

### Junior-Level Answer

An optional is a value that may contain something or may be nil.

```swift
var name: String?
```

### Mid-Level Answer

Optionals are Swift's safe way to represent missing values. The compiler forces us to unwrap them before using the wrapped value.

### Senior-Level Answer

Optionals are domain modeling. A value should be optional only when absence is valid. Using optionals correctly makes APIs honest. Overusing them can create unclear state, while underusing them can hide missing values behind fake defaults.

## Question 2: How Is Optional Related To Enum?

Answer:

Optional is conceptually an enum:

```swift
enum Optional<Wrapped> {
    case some(Wrapped)
    case none
}
```

That is why optionals can be pattern matched with `.some` and `.none`.

## Question 3: What Is Optional Binding?

Answer:

Optional binding safely unwraps an optional.

```swift
if let name {
    print(name)
}
```

Use `if let` for local branch logic and `guard let` when the value is required for the rest of the function.

## Question 4: What Is Optional Chaining?

Answer:

Optional chaining uses `?.` to safely access properties or methods on optionals.

```swift
let city = user.address?.city
```

If `address` is nil, `city` is nil.

Senior note:

Long optional chains can indicate that UI code knows too much about nested API structure. Consider mapping to view models.

## Question 5: What Is Nil-Coalescing?

Answer:

Nil-coalescing uses `??` to provide a fallback value.

```swift
let title = user.nickname ?? user.fullName
```

Senior note:

Use fallbacks only when fallback is valid. Do not hide missing required IDs, tokens, or permissions with empty strings or false values.

## Question 6: Why Is Force Unwrap Risky?

Answer:

Force unwrap uses `!`. If the optional is nil, the app crashes.

```swift
let url = URL(string: input)!
```

This is unsafe if `input` can be invalid.

Senior note:

Force unwrap is a deliberate assertion. It should be rare and backed by a strong invariant.

## Question 7: What Is An Implicitly Unwrapped Optional?

Answer:

An implicitly unwrapped optional is written as `Type!`. It can be used without manual unwrapping, but crashes if nil.

```swift
@IBOutlet weak var titleLabel: UILabel!
```

Senior note:

IUOs are lifecycle escape hatches, mostly useful for UIKit outlets or controlled test setup. Avoid them for ordinary app state.

## Question 8: What Is Optional Pattern Matching?

Answer:

Optional pattern matching checks optionals using patterns like `.some`, `.none`, or `value?`.

```swift
for case let value? in values {
    print(value)
}
```

Use it when optionals are part of a larger pattern.

## Real iOS Use Case 1: API Model

```swift
struct ArticleResponse: Decodable {
    let id: String
    let title: String
    let subtitle: String?
    let imageURL: URL?
}
```

Discussion:

- `id` and `title` are required.
- `subtitle` and `imageURL` may be missing.
- The model communicates backend reality.

## Real iOS Use Case 2: View Model Mapping

```swift
struct ArticleRowViewModel {
    let title: String
    let subtitle: String
    let imageURL: URL?
}

func makeRow(from response: ArticleResponse) -> ArticleRowViewModel {
    ArticleRowViewModel(
        title: response.title,
        subtitle: response.subtitle ?? "No subtitle",
        imageURL: response.imageURL
    )
}
```

Discussion:

The view model converts optional display text into a non-optional string, while keeping `imageURL` optional because the UI may show a placeholder.

## Real iOS Use Case 3: View State

Weak model:

```swift
var user: User?
var errorMessage: String?
var isLoading: Bool = false
```

Better model:

```swift
enum ProfileState {
    case loading
    case loaded(User)
    case empty
    case failed(String)
}
```

Discussion:

The enum prevents invalid state combinations.

## Real iOS Use Case 4: Delegate

```swift
weak var delegate: LoginDelegate?

delegate?.didFinishLogin()
```

Discussion:

The delegate is optional because the object may not have a listener.

## Junior-Level Rapid Revision

- Optional means value or nil.
- `String?` is optional string.
- `if let` unwraps safely.
- `guard let` unwraps and exits if nil.
- `?.` is optional chaining.
- `??` gives a default.
- `!` force unwrap can crash.

## Mid-Level Rapid Revision

- Use optionals when absence is meaningful.
- Do not use empty strings to represent missing values.
- Use nil-coalescing for harmless display fallback.
- Use guard for required values.
- Avoid force unwrap for API, user input, and persistence data.
- Use optional delegates with `weak` in UIKit patterns.

## Senior-Level Rapid Revision

- Optionals are domain modeling, not just syntax.
- Too many optionals can indicate missing state modeling.
- Required API data should fail validation if absent.
- View models can convert optional API fields into UI-ready values.
- IUOs should be limited to lifecycle-driven framework cases.
- Force unwrap should be treated as an assertion.
- Optional chaining is safe but long chains can hide architecture issues.

## Coding Exercise 1: Safe URL Builder

```swift
enum URLError: Error {
    case invalidURL
}

func makeURL(from text: String) throws -> URL {
    guard let url = URL(string: text) else {
        throw URLError.invalidURL
    }

    return url
}
```

## Coding Exercise 2: Display Name

```swift
func displayName(firstName: String?, lastName: String?) -> String {
    let first = firstName?.trimmingCharacters(in: .whitespaces) ?? ""
    let last = lastName?.trimmingCharacters(in: .whitespaces) ?? ""
    let fullName = "\(first) \(last)".trimmingCharacters(in: .whitespaces)

    return fullName.isEmpty ? "Guest" : fullName
}
```

## Coding Exercise 3: Optional Pattern Matching

```swift
func printValidScores(_ scores: [Int?]) {
    for case let score? in scores where score >= 0 {
        print(score)
    }
}
```

## Coding Exercise 4: Refactor Optional State

Before:

```swift
struct ScreenModel {
    var data: [String]?
    var error: String?
    var isLoading: Bool
}
```

After:

```swift
enum ScreenState {
    case loading
    case loaded([String])
    case empty
    case failed(String)
}
```

## Common Mistakes

### Force Unwrapping API Data

```swift
let id = response.id!
```

Use validation instead.

### Using Empty String For Missing Data

```swift
let middleName = ""
```

Use optional when absence is meaningful.

### Overusing IUOs

```swift
var selectedUser: User!
```

Use normal optional or explicit state.

### Hiding Required Missing Values

```swift
let token = authToken ?? ""
```

Authentication token absence should usually be an error.

## Final Interview Questions

1. What is an optional?
2. Why does Swift have optionals?
3. How is optional related to enum?
4. What is optional binding?
5. What is the difference between `if let` and `guard let`?
6. What does optional chaining return?
7. What does nil-coalescing do?
8. Why is force unwrap risky?
9. When can force unwrap be acceptable?
10. What is an implicitly unwrapped optional?
11. Why are IBOutlets often IUOs?
12. What is optional pattern matching?
13. How should missing API fields be handled?
14. When should optionals be replaced by an enum state?
15. How do optionals improve iOS app safety?

