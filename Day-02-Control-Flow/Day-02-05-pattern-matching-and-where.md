# Day 2: Pattern Matching And Where Clauses

## What You Will Learn

- What pattern matching means in Swift
- How patterns appear in `switch`, `if case`, `guard case`, and `for case`
- How `where` adds conditions
- How pattern matching improves enum-heavy iOS code
- Junior to senior-level interview explanations

## Core Idea

Pattern matching checks whether a value fits a specific shape.

You already use pattern matching when switching over enums:

```swift
enum ResultState {
    case success(String)
    case failure(Error)
}

let state = ResultState.success("Loaded")

switch state {
case .success(let message):
    print(message)
case .failure(let error):
    print(error)
}
```

The case `.success(let message)` matches only success values and extracts the associated message.

## If Case

Use `if case` when you only care about one pattern.

```swift
enum ViewState {
    case loading
    case content(String)
    case error(String)
}

let state = ViewState.content("Profile loaded")

if case .content(let message) = state {
    print(message)
}
```

This is cleaner than a full `switch` when only one case matters.

## Guard Case

Use `guard case` when a specific pattern is required to continue.

```swift
func render(state: ViewState) {
    guard case .content(let message) = state else {
        print("Cannot render content")
        return
    }

    print("Render: \(message)")
}
```

This is similar to `guard let`, but for enum cases and patterns.

## For Case

Use `for case` to iterate only matching values.

```swift
let states: [ViewState] = [
    .loading,
    .content("Home"),
    .error("No internet"),
    .content("Profile")
]

for case .content(let title) in states {
    print(title)
}
```

This prints only content values.

## Optional Pattern Matching

Optionals are enums internally:

```swift
enum Optional<Wrapped> {
    case some(Wrapped)
    case none
}
```

You can pattern match them:

```swift
let name: String? = "Riya"

if case .some(let value) = name {
    print(value)
}
```

Most of the time, use `if let`:

```swift
if let name {
    print(name)
}
```

But optional pattern matching becomes useful in advanced cases.

## Where Clauses In Switch

```swift
let transaction = (amount: 2500, category: "Travel")

switch transaction {
case let (amount, category) where amount > 2000 && category == "Travel":
    print("Large travel expense")
case let (amount, _) where amount > 2000:
    print("Large expense")
default:
    print("Normal expense")
}
```

`where` lets you add boolean rules to a pattern.

## Where In Loops

```swift
let numbers = [1, 2, 3, 4, 5, 6]

for number in numbers where number.isMultiple(of: 2) {
    print(number)
}
```

Use this for simple filtering. For complex filtering, consider a named function or `filter`.

## Tuple Pattern Matching

```swift
let locationPermission = (isGranted: true, isPrecise: false)

switch locationPermission {
case (true, true):
    print("Precise location available")
case (true, false):
    print("Approximate location available")
case (false, _):
    print("Location denied")
}
```

Tuple matching is useful for small combinations of values. If the combination becomes domain-heavy, prefer an enum or struct.

## Pattern Matching With Ranges

```swift
let batteryLevel = 18

switch batteryLevel {
case 0...20:
    print("Low battery")
case 21...80:
    print("Normal battery")
case 81...100:
    print("High battery")
default:
    print("Invalid battery level")
}
```

## Pattern Matching With Type Casting

```swift
let value: Any = "Swift"

switch value {
case let text as String:
    print("String: \(text)")
case let number as Int:
    print("Int: \(number)")
default:
    print("Unknown type")
}
```

Use type-casting switches sparingly. Strong modeling is usually better than passing around `Any`.

## Real iOS Use Cases

### Handling Screen State

```swift
enum ScreenState {
    case idle
    case loading
    case loaded(items: [String])
    case failed(message: String)
}

let screenState = ScreenState.loaded(items: ["A", "B"])

if case .loaded(let items) = screenState, !items.isEmpty {
    print("Show \(items.count) items")
}
```

### Filtering Analytics Events

```swift
enum AnalyticsEvent {
    case screenView(name: String)
    case buttonTap(id: String)
    case purchase(amount: Decimal)
}

let events: [AnalyticsEvent] = [
    .screenView(name: "Home"),
    .buttonTap(id: "pay_now"),
    .purchase(amount: 499)
]

for case .buttonTap(let id) in events {
    print("Tapped: \(id)")
}
```

### Navigation Routes

```swift
enum Route {
    case profile(userId: String)
    case settings
    case transaction(id: String)
}

func handle(route: Route) {
    switch route {
    case .profile(let userId):
        print("Open profile \(userId)")
    case .settings:
        print("Open settings")
    case .transaction(let id):
        print("Open transaction \(id)")
    }
}
```

## Junior-Level Interview Answer

Question: What is pattern matching?

Answer:

Pattern matching checks whether a value matches a specific pattern. In Swift, it is commonly used in `switch` statements, especially with enums and associated values.

## Mid-Level Interview Answer

Question: What is `if case` used for?

Answer:

`if case` is used when we want to check one specific pattern without writing a full switch. It is useful for checking one enum case and extracting its associated value.

## Senior-Level Interview Answer

Question: How does pattern matching help with app architecture?

Answer:

Pattern matching helps model finite state and domain events clearly. When states are represented as enums with associated values, `switch`, `if case`, and `guard case` let the compiler enforce correct handling. This reduces invalid state combinations, removes stringly typed logic, and improves maintainability in view models, reducers, navigation, analytics, and networking layers.

## Quick Interview Notes

- Pattern matching checks a value's shape.
- `switch` is the most common pattern matching tool.
- `if case` is useful for one pattern.
- `guard case` is useful when one pattern is required to continue.
- `for case` filters matching values during iteration.
- `where` adds extra conditions to a pattern.
- Enums with associated values are a major Swift pattern matching use case.

## Points To Remember

- Prefer enums over strings for finite states.
- Avoid overusing tuple matching for complex domain logic.
- Use `where` to keep patterns expressive but not overloaded.
- Pattern matching makes impossible states harder to represent.
- Senior Swift code often uses pattern matching to make state handling explicit.

## Practice Questions

1. What is pattern matching in Swift?
2. What is the difference between `switch` and `if case`?
3. When would you use `guard case`?
4. How does `where` work with pattern matching?
5. Why are enums powerful with pattern matching?

