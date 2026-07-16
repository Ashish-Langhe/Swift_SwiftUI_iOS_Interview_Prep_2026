# Day 1: Variables And Constants

## What You Will Learn

- Difference between `let` and `var`
- Why Swift encourages constants
- How type inference works with stored values
- Naming conventions
- Scope and lifetime basics
- Mutability in real iOS code
- Interview-ready explanations

## Core Idea

Swift stores values using two main declaration keywords:

- `let`: creates a constant. Its value cannot be changed after assignment.
- `var`: creates a variable. Its value can be changed after assignment.

Use `let` by default. Use `var` only when the value must change.

```swift
let appName = "Interview Prep"
var completedLessons = 0

completedLessons += 1

print(appName)
print(completedLessons)
```

Here, `appName` should not change while the app is running, so it is a constant. `completedLessons` changes as the user studies, so it is a variable.

## Why Swift Prefers `let`

Constants make code safer and easier to reason about.

```swift
let userId = "A1029"
let createdAt = Date()
```

When another developer sees `let`, they know the value will not be reassigned later. This reduces bugs, especially in larger codebases.

## Reassignment

`var` allows reassignment:

```swift
var score = 10
score = 20
```

`let` does not:

```swift
let maxAttempts = 3
// maxAttempts = 4
// Error: Cannot assign to value: 'maxAttempts' is a 'let' constant
```

## Constants With Reference Types

This is a common interview trap.

If a class instance is stored with `let`, the reference cannot point to another object, but the object's mutable properties may still change.

```swift
final class UserProfile {
    var name: String

    init(name: String) {
        self.name = name
    }
}

let profile = UserProfile(name: "Aarav")
profile.name = "Aarav Sharma" // Allowed

// profile = UserProfile(name: "Neha")
// Error: Cannot assign to value: 'profile' is a 'let' constant
```

For structs, changing a property usually means changing the whole value:

```swift
struct Settings {
    var isDarkModeEnabled: Bool
}

let settings = Settings(isDarkModeEnabled: false)
// settings.isDarkModeEnabled = true
// Error: Cannot assign to property: 'settings' is a 'let' constant
```

## Explicit Type Declaration

Swift can infer the type:

```swift
let name = "Priya"        // String
let age = 28              // Int
let rating = 4.8          // Double
let isPremium = true      // Bool
```

You can also write the type explicitly:

```swift
let name: String = "Priya"
let age: Int = 28
let rating: Double = 4.8
let isPremium: Bool = true
```

Use explicit types when:

- The type is not obvious.
- You want a specific numeric type.
- You are declaring a variable without an initial value.
- The explicit type improves readability.

```swift
var selectedUserId: String?
var retryCount: Int = 0
let price: Decimal = 199.99
```

## Declaring Multiple Values

```swift
let x = 10, y = 20
var firstName = "Anika", lastName = "Rao"
```

This is valid, but for interview and production readability, separate declarations are usually clearer:

```swift
let x = 10
let y = 20
```

## Naming Rules

Good Swift names are descriptive and use lower camel case.

```swift
let userName = "Kabir"
var unreadMessageCount = 5
let maximumLoginAttempts = 3
```

Avoid unclear names:

```swift
let x = "Kabir"
var cnt = 5
```

Short names are fine only in very small scopes:

```swift
for i in 1...5 {
    print(i)
}
```

## Scope

A value exists only inside the scope where it is declared.

```swift
func showWelcomeMessage() {
    let message = "Welcome back"
    print(message)
}

// print(message)
// Error: Cannot find 'message' in scope
```

Nested scopes can access outer values:

```swift
let taxRate = 0.18

func finalPrice(for amount: Double) -> Double {
    let tax = amount * taxRate
    return amount + tax
}
```

## Shadowing

Shadowing means declaring a new value with the same name in an inner scope.

```swift
let status = "Logged out"

func login() {
    let status = "Logged in"
    print(status)
}

print(status)
login()
```

Output:

```text
Logged out
Logged in
```

Shadowing is legal, but use it carefully. It can make code confusing when overused.

## Real iOS Use Cases

### Screen Title

```swift
let navigationTitle = "Expenses"
```

A title usually does not need reassignment inside the same view body.

### Loading State

```swift
var isLoading = false

func loadData() {
    isLoading = true
    // Fetch data
    isLoading = false
}
```

Loading state changes, so it is a variable.

### API Configuration

```swift
let baseURL = URL(string: "https://api.example.com")!
let timeoutInterval: TimeInterval = 30
```

Configuration values should usually be constants.

### SwiftUI State

```swift
import SwiftUI

struct CounterView: View {
    @State private var count = 0

    var body: some View {
        Button("Count: \(count)") {
            count += 1
        }
    }
}
```

`count` is a variable because the UI state changes over time.

## Important Swift 6.x Context

Swift 6 made data-race safety a major focus. This does not change the basic syntax of `let` and `var`, but it makes mutability more important.

In concurrent code, mutable shared state is risky:

```swift
var totalDownloads = 0

func downloadFinished() {
    totalDownloads += 1
}
```

If many tasks call this at the same time, the value may be updated unsafely. Modern Swift encourages safer patterns:

```swift
actor DownloadCounter {
    private var total = 0

    func increment() {
        total += 1
    }

    func value() -> Int {
        total
    }
}
```

Interview explanation:

`let` reduces accidental mutation. `actor` protects mutable state that must be shared across concurrent tasks.

## Common Mistakes

### Using `var` everywhere

```swift
var apiKey = "abc123"
```

Better:

```swift
let apiKey = "abc123"
```

### Force-unwrapping because a variable was not initialized

```swift
var username: String!
```

Better:

```swift
var username: String?
```

Or initialize it:

```swift
let username = "Guest"
```

### Making mutable global state

```swift
var currentUserName = "Guest"
```

Prefer dependency injection, app state containers, actors, or SwiftUI state tools depending on the use case.

## Quick Interview Notes

- `let` creates a constant; `var` creates a variable.
- Prefer `let` unless reassignment is required.
- `let` with a class prevents reassignment of the reference, not mutation of the object's internal mutable properties.
- `let` with a struct prevents mutation of stored properties.
- Swift is strongly typed, but type inference reduces boilerplate.
- Mutability matters more in Swift 6 because concurrency checking is stricter.
- Avoid unnecessary global mutable state.

## Points To Remember

- Good Swift code communicates intent through mutability.
- Constants improve safety, readability, and optimization opportunities.
- Variables should represent values that genuinely change.
- In SwiftUI, `@State var` is used for view-local mutable state.
- In concurrent code, protect shared mutable state with actors or other safe isolation strategies.

## Practice Questions

1. What is the difference between `let` and `var`?
2. Can you mutate a property of a class instance stored in a `let` constant?
3. Why does Swift recommend constants by default?
4. What happens if you try to modify a property of a struct stored in a `let` constant?
5. Why is mutable global state dangerous in concurrent Swift code?

