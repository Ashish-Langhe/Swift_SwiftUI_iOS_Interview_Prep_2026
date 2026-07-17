# Day 17: `private`

## What `private` Means

`private` is the most restrictive normal access level in Swift.

A `private` declaration can be used only inside the declaration scope where it is written, plus extensions of that declaration in the same file.

```swift
final class LoginViewModel {
    private var failedAttemptCount = 0

    func login(email: String, password: String) {
        if validate(email: email, password: password) {
            failedAttemptCount = 0
        } else {
            failedAttemptCount += 1
        }
    }

    private func validate(email: String, password: String) -> Bool {
        !email.isEmpty && password.count >= 8
    }
}
```

`failedAttemptCount` and `validate` are implementation details of `LoginViewModel`.

## Beginner Mental Model

Use `private` when you want to say:

```text
Only this type or scope should know this exists.
```

It protects a type's internal state from accidental external mutation.

```swift
struct Cart {
    private(set) var items: [CartItem] = []

    mutating func add(_ item: CartItem) {
        items.append(item)
    }
}
```

Here:

- Other code can read `items`.
- Only `Cart` can change `items`.
- External code must use `add(_:)`, so the type controls its own rules.

## `private` vs `private(set)`

`private(set)` is extremely common in iOS code.

```swift
final class DownloadViewModel {
    private(set) var progress: Double = 0

    func updateProgress(_ value: Double) {
        progress = min(max(value, 0), 1)
    }
}
```

This means:

- The property can be read using the property's normal access level.
- The setter is private.

This is useful when a view needs to display state but should not mutate it directly.

## `private` In Extensions

Swift allows a `private` member to be used from extensions of the same type in the same file.

```swift
final class ProfileViewModel {
    private var cache: [String: User] = [:]
}

extension ProfileViewModel {
    func cachedUser(id: String) -> User? {
        cache[id]
    }
}
```

This lets you organize code with extensions without losing access to private implementation.

## Real iOS Use Cases

Use `private` for:

- Internal validation helpers
- Mutable backing state
- Dependency objects that should not leak out
- Formatting helpers
- View-building helpers in SwiftUI
- UIKit setup methods
- Constants used only by one type

```swift
final class ProfileViewController: UIViewController {
    private let avatarImageView = UIImageView()
    private let nameLabel = UILabel()

    override func viewDidLoad() {
        super.viewDidLoad()
        configureLayout()
    }

    private func configureLayout() {
        view.addSubview(avatarImageView)
        view.addSubview(nameLabel)
    }
}
```

The view controller exposes behavior, not its individual UI pieces.

## SwiftUI Example

```swift
struct SettingsView: View {
    @State private var isNotificationsEnabled = false

    var body: some View {
        Form {
            Toggle("Notifications", isOn: $isNotificationsEnabled)
            resetButton
        }
    }

    private var resetButton: some View {
        Button("Reset") {
            isNotificationsEnabled = false
        }
    }
}
```

`@State private` is the standard pattern because the state belongs to the view.

## Why `private` Matters

Access control is not only security. It is design.

Good `private` usage:

- Reduces accidental coupling
- Makes refactoring safer
- Shows which members are real API
- Prevents invalid state changes
- Makes code easier to review

```swift
struct BankAccount {
    private(set) var balance: Decimal = 0

    mutating func deposit(_ amount: Decimal) {
        guard amount > 0 else { return }
        balance += amount
    }

    mutating func withdraw(_ amount: Decimal) -> Bool {
        guard amount > 0, balance >= amount else { return false }
        balance -= amount
        return true
    }
}
```

External code cannot do this:

```swift
// account.balance = -10
```

That is exactly the point.

## Advanced Design Notes

At senior level, `private` is about controlling invariants.

An invariant is a rule that should always remain true.

Examples:

- A progress value should stay between `0` and `1`.
- A cart total should match cart items.
- A logged-in session should have a valid token.
- A view model should publish state but own mutations.

If external code can mutate every property directly, your type cannot protect its invariants.

## Common Mistakes

Mistake:

```swift
final class UserSession {
    var token: String?
}
```

Better:

```swift
final class UserSession {
    private(set) var token: String?

    func updateToken(_ token: String) {
        self.token = token
    }

    func clear() {
        token = nil
    }
}
```

The second version communicates ownership.

## Modern Swift 6.x Notes

The meaning of `private` itself is stable in modern Swift.

What changed around modern Swift is the importance of isolation:

- Swift concurrency encourages clear ownership of mutable state.
- `private` helps keep mutation localized.
- Actor isolation and access control solve different problems.
- `private` does not make shared mutable state thread-safe.

```swift
actor TokenStore {
    private var token: String?

    func update(_ token: String) {
        self.token = token
    }

    func currentToken() -> String? {
        token
    }
}
```

Here:

- `private` hides the stored property.
- `actor` protects access across concurrency domains.

## Junior Interview Answer

`private` restricts access to the current declaration scope and same-file extensions of that declaration. It is used to hide implementation details.

## Mid-Level Interview Answer

I use `private` for helper methods, stored properties, and state that should only be modified by the owning type. `private(set)` is useful when outside code can read a value but only the type can mutate it.

## Senior Interview Answer

`private` is a design tool for preserving invariants and minimizing API surface. It allows a type to expose behavior instead of data. In larger codebases, heavy use of `private` makes refactoring safer because fewer declarations become accidental dependencies. It should be combined with dependency injection, protocol boundaries, and concurrency isolation where needed.

## Points To Remember

- `private` is more restrictive than `fileprivate`.
- Use `private(set)` for read-public, write-private state.
- `private` helps refactoring.
- `private` does not mean encrypted or secure.
- `private` does not solve threading by itself.

## Practice

1. Create a `Cart` type where items can be read but only modified through methods.
2. Convert public helper methods in a view model into private helpers.
3. Explain why `@State private var` is common in SwiftUI.
4. Find one type in an app where direct property mutation can create invalid state.
