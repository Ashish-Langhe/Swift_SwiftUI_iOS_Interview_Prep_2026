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

