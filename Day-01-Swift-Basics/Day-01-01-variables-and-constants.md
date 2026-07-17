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

## Deep Dive Expansion

### Why This Topic Matters In Real Apps

Variables And Constants is not only a syntax topic. In production Swift, it affects type safety, mutability, compiler feedback, and readable beginner code. A junior developer should be able to recognize the syntax and explain the basic behavior. A mid-level developer should know when the feature improves clarity or safety. A senior developer should understand how it changes API design, testability, performance, or long-term maintenance.

In iOS work, this usually appears while building a small profile form, parsing values, and keeping invalid states out of your model. The important question is not "Can I write the syntax?" The important question is "Does this code make invalid states harder, make ownership clearer, and make future changes safer?"

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

Use this checklist when applying Variables And Constants in an app feature:

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

- Prefer code that communicates intent at the call site, not only code that compiles.
- When a feature grows, revisit whether the type still owns the right responsibilities.

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

1. Write a minimal example that shows Variables And Constants correctly.
2. Write a second example that demonstrates a common mistake.
3. Refactor the mistake into a safer version.
4. Explain the difference between the beginner solution and the production-ready solution.
5. Add one test case or reasoning check that proves the behavior.

## Applied Day-Level Example

The following example is not the only way to use Variables And Constants, but it shows the kind of production shape you should connect this topic to:

```swift
struct SignupDraft {
    var email: String
    var password: String
    let createdAt: Date

    var isReadyToSubmit: Bool {
        !email.isEmpty && password.count >= 8
    }
}

var draft = SignupDraft(email: "", password: "", createdAt: Date())
draft.email = "user@example.com"
draft.password = "12345678"

if draft.isReadyToSubmit {
    print("Submit enabled")
}
```

This example connects basics to real form state. `var` marks values the user edits, `let` protects creation metadata, and the computed property gives the UI a readable decision point.

### How To Study This Example

- Identify which part is syntax and which part is design.
- Ask what invalid state the example prevents.
- Ask what would need to change if the feature became async, generic, public, or shared across modules.
- Rewrite the example with one deliberate bug, then explain how a reviewer should catch it.
- Turn the example into a two-minute interview explanation with tradeoffs.

