# Day 4: Optional Binding

## Core Idea

Optional binding safely unwraps an optional into a non-optional value.

```swift
let name: String? = "Aarav"

if let name {
    print("Hello, \(name)")
}
```

Inside the `if` block, `name` is a normal `String`.

## Old And Modern Syntax

Older style:

```swift
if let name = name {
    print(name)
}
```

Modern shorthand:

```swift
if let name {
    print(name)
}
```

The shorthand is common when the unwrapped variable should use the same name.

## If Let

Use `if let` when you want to run code only if a value exists.

```swift
let profileImageURL: URL? = URL(string: "https://example.com/avatar.png")

if let profileImageURL {
    print("Load image from \(profileImageURL)")
} else {
    print("Show placeholder")
}
```

## Guard Let

Use `guard let` when the value is required for the rest of the function.

```swift
func openProfile(userId: String?) {
    guard let userId else {
        print("Missing user id")
        return
    }

    print("Open profile \(userId)")
}
```

After the guard, `userId` is non-optional.

## Multiple Optional Bindings

```swift
let email: String? = "user@example.com"
let password: String? = "secret123"

if let email, let password {
    print("Login with \(email), password length \(password.count)")
}
```

Add conditions:

```swift
if let email, email.contains("@"), let password, password.count >= 8 {
    print("Valid login input")
}
```

## Binding With Different Names

```swift
let rawName: String? = " Meera "

if let unwrappedName = rawName {
    let cleanName = unwrappedName.trimmingCharacters(in: .whitespaces)
    print(cleanName)
}
```

Different names can help when transformation or clarity matters.

## Optional Binding In Loops

```swift
let values: [String?] = ["A", nil, "B"]

for value in values {
    if let value {
        print(value)
    }
}
```

Better with `compactMap`:

```swift
for value in values.compactMap({ $0 }) {
    print(value)
}
```

## Real iOS Use Cases

### ViewModel Action

```swift
func payNow(selectedCardId: String?) {
    guard let selectedCardId else {
        print("Ask user to select a card")
        return
    }

    print("Start payment with card \(selectedCardId)")
}
```

### API URL

```swift
func fetch(from urlString: String) {
    guard let url = URL(string: urlString) else {
        print("Invalid URL")
        return
    }

    print("Request \(url)")
}
```

### SwiftUI State

```swift
import SwiftUI

struct ProfileNameView: View {
    let name: String?

    var body: some View {
        if let name {
            Text(name)
        } else {
            Text("Guest")
        }
    }
}
```

## Junior-Level Interview Answer

Optional binding safely checks whether an optional has a value. If it has a value, Swift unwraps it.

## Mid-Level Interview Answer

I use `if let` when the unwrapped value is needed inside one branch, and `guard let` when the value is required for the rest of the function.

## Senior-Level Interview Answer

Optional binding makes absence handling explicit. In production code, I choose `if let`, `guard let`, nil-coalescing, or optional chaining based on what the absence means. If too many bindings appear everywhere, it may mean the model is too optional-heavy or needs a better state representation.

## Quick Interview Notes

- `if let` unwraps inside the `if` block.
- `guard let` unwraps for the rest of the scope.
- Modern shorthand is `if let name`.
- Optional binding avoids force unwrap crashes.
- Multiple optionals can be bound in one condition.

## Practice Questions

1. What is optional binding?
2. What is the difference between `if let` and `guard let`?
3. What does `if let name` mean?
4. How do you bind multiple optionals?
5. When can too much optional binding indicate a design issue?

