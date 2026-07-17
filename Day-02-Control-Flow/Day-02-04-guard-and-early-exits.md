# Day 2: Guard And Early Exits

## What You Will Learn

- What `guard` does
- How `guard` improves readability
- How to unwrap optionals using `guard let`
- How early exits reduce nesting
- How to explain `guard` from junior to senior level
- Real iOS use cases

## Core Idea

`guard` checks that a condition is true. If the condition is false, the `else` block must exit the current scope.

```swift
func greet(name: String?) {
    guard let name else {
        print("No name found")
        return
    }

    print("Hello, \(name)")
}
```

After `guard let`, the unwrapped value is available after the guard statement.

## Basic Syntax

```swift
guard condition else {
    return
}
```

The `else` block must exit using one of:

- `return`
- `throw`
- `break`
- `continue`
- `fatalError()`

Example:

```swift
func submitForm(email: String) {
    guard !email.isEmpty else {
        print("Email required")
        return
    }

    print("Submitting form")
}
```

## Guard Let

Use `guard let` to unwrap optionals and continue with non-optional values.

```swift
func showProfile(userId: String?) {
    guard let userId else {
        print("Missing user id")
        return
    }

    print("Load profile for \(userId)")
}
```

Inside the rest of the function, `userId` is a non-optional `String`.

## Multiple Guard Conditions

```swift
func login(email: String?, password: String?) {
    guard let email, !email.isEmpty else {
        print("Invalid email")
        return
    }

    guard let password, password.count >= 8 else {
        print("Invalid password")
        return
    }

    print("Login with \(email)")
}
```

This keeps the happy path flat and readable.

## Guard With Throwing Functions

```swift
enum LoginError: Error {
    case missingEmail
    case weakPassword
}

func validate(email: String?, password: String?) throws {
    guard let email, !email.isEmpty else {
        throw LoginError.missingEmail
    }

    guard let password, password.count >= 8 else {
        throw LoginError.weakPassword
    }

    print("Validated \(email)")
}
```

Throwing guards are common in validation and parsing.

## Guard In Loops

Use `continue` when one item should be skipped.

```swift
let names = ["Aarav", "", "Meera"]

for name in names {
    guard !name.isEmpty else {
        continue
    }

    print(name)
}
```

Use `break` when the loop should stop.

```swift
for name in names {
    guard !name.isEmpty else {
        break
    }

    print(name)
}
```

## Guard Vs If Let

Use `if let` when the unwrapped value is only needed inside one branch.

```swift
if let imageURL {
    print("Load image from \(imageURL)")
} else {
    print("Show placeholder")
}
```

Use `guard let` when the value is required for the rest of the function.

```swift
guard let imageURL else {
    print("Show placeholder")
    return
}

print("Load image from \(imageURL)")
```

## Early Exit Pattern

Without `guard`, code can become deeply nested.

```swift
func process(user: User?) {
    if let user {
        if user.isActive {
            if user.hasPermission {
                print("Process user")
            }
        }
    }
}
```

With `guard`:

```swift
func process(user: User?) {
    guard let user else { return }
    guard user.isActive else { return }
    guard user.hasPermission else { return }

    print("Process user")
}
```

The second version makes requirements obvious.

## Real iOS Use Cases

### Table View Cell Configuration

```swift
func configureCell(with user: User?) {
    guard let user else {
        return
    }

    print("Configure cell for \(user.name)")
}
```

### Network Response Validation

```swift
func handleResponse(data: Data?, response: URLResponse?) throws {
    guard let data else {
        throw NetworkError.emptyData
    }

    guard let httpResponse = response as? HTTPURLResponse else {
        throw NetworkError.invalidResponse
    }

    guard 200..<300 ~= httpResponse.statusCode else {
        throw NetworkError.badStatusCode(httpResponse.statusCode)
    }

    print("Decode \(data.count) bytes")
}
```

### ViewModel Action

```swift
@MainActor
final class PaymentViewModel {
    var selectedCardId: String?

    func payNow() {
        guard let selectedCardId else {
            print("Ask user to select card")
            return
        }

        print("Start payment with card \(selectedCardId)")
    }
}
```

## Junior-Level Interview Answer

Question: What is `guard` in Swift?

Answer:

`guard` checks a condition that must be true for the code to continue. If the condition is false, the `else` block must exit the current scope using `return`, `throw`, `break`, or `continue`.

## Mid-Level Interview Answer

Question: Why use `guard` instead of `if`?

Answer:

I use `guard` when a condition is required for the rest of the function. It avoids deep nesting and keeps the main logic in the normal left-aligned path. It is especially useful for validation, optional unwrapping, and early exits.

## Senior-Level Interview Answer

Question: How does `guard` improve production code quality?

Answer:

`guard` makes preconditions explicit. In production code, this improves readability, reduces nesting, and makes failure paths visible. For API parsing, permissions, form validation, and view model actions, guard statements communicate "this must be true before continuing." It also scopes unwrapped values to the remainder of the function, which reduces optional noise and makes later logic safer.

## Quick Interview Notes

- `guard` requires an exit in its `else` block.
- `guard let` unwraps optionals for the rest of the scope.
- Use `guard` for required conditions.
- Use `if` for branching where both paths are meaningful.
- `guard` reduces nesting and highlights failure paths.

## Points To Remember

- Do not use `guard` just because it looks advanced.
- Use `guard` when the rest of the scope cannot continue safely.
- Keep guard failure blocks short and clear.
- Prefer meaningful thrown errors over silent returns in important flows.
- Guard statements are excellent for input validation.

## Practice Questions

1. Why must the `else` block of `guard` exit?
2. What is the difference between `if let` and `guard let`?
3. Can you use `throw` inside a guard else block?
4. How does `guard` reduce nesting?
5. Give one iOS example where `guard` is useful.

## Deep Dive Expansion

### Why This Topic Matters In Real Apps

Guard And Early Exits is not only a syntax topic. In production Swift, it affects branch clarity, exhaustiveness, early exits, and reducing nested logic. A junior developer should be able to recognize the syntax and explain the basic behavior. A mid-level developer should know when the feature improves clarity or safety. A senior developer should understand how it changes API design, testability, performance, or long-term maintenance.

In iOS work, this usually appears while validating form input, handling API states, and routing user actions. The important question is not "Can I write the syntax?" The important question is "Does this code make invalid states harder, make ownership clearer, and make future changes safer?"

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

Use this checklist when applying Guard And Early Exits in an app feature:

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

1. Write a minimal example that shows Guard And Early Exits correctly.
2. Write a second example that demonstrates a common mistake.
3. Refactor the mistake into a safer version.
4. Explain the difference between the beginner solution and the production-ready solution.
5. Add one test case or reasoning check that proves the behavior.

## Applied Day-Level Example

The following example is not the only way to use Guard And Early Exits, but it shows the kind of production shape you should connect this topic to:

```swift
enum CheckoutState {
    case emptyCart
    case needsAddress
    case readyToPay(total: Decimal)
    case processing
    case failed(String)
}

func primaryActionTitle(for state: CheckoutState) -> String {
    switch state {
    case .emptyCart:
        return "Continue Shopping"
    case .needsAddress:
        return "Add Address"
    case .readyToPay:
        return "Pay Now"
    case .processing:
        return "Processing"
    case .failed:
        return "Try Again"
    }
}
```

This is the production mindset for control flow: branch on domain state, keep each case intentional, and let the compiler force you to handle new states.

### How To Study This Example

- Identify which part is syntax and which part is design.
- Ask what invalid state the example prevents.
- Ask what would need to change if the feature became async, generic, public, or shared across modules.
- Rewrite the example with one deliberate bug, then explain how a reviewer should catch it.
- Turn the example into a two-minute interview explanation with tradeoffs.

