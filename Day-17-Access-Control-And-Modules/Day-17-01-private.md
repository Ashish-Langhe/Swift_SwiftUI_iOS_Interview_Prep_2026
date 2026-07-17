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

## Deep Dive Expansion

### Why This Topic Matters In Real Apps

private is not only a syntax topic. In production Swift, it affects API surface, encapsulation, package boundaries, framework evolution, and source compatibility. A junior developer should be able to recognize the syntax and explain the basic behavior. A mid-level developer should know when the feature improves clarity or safety. A senior developer should understand how it changes API design, testability, performance, or long-term maintenance.

In iOS work, this usually appears while feature modules, design systems, SDK entry points, and internal helpers. The important question is not "Can I write the syntax?" The important question is "Does this code make invalid states harder, make ownership clearer, and make future changes safer?"

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

Use this checklist when applying private in an app feature:

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

- Start restrictive and widen access only when another boundary genuinely needs it.
- Treat public and open declarations as compatibility promises, not convenience keywords.

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

1. Write a minimal example that shows private correctly.
2. Write a second example that demonstrates a common mistake.
3. Refactor the mistake into a safer version.
4. Explain the difference between the beginner solution and the production-ready solution.
5. Add one test case or reasoning check that proves the behavior.

## Applied Day-Level Example

The following example is not the only way to use private, but it shows the kind of production shape you should connect this topic to:

```swift
public struct LoginFeatureView: View {
    private let configuration: LoginConfiguration

    public init(configuration: LoginConfiguration) {
        self.configuration = configuration
    }

    public var body: some View {
        LoginContentView(configuration: configuration)
    }
}

private struct LoginContentView: View {
    let configuration: LoginConfiguration
    var body: some View { Text(configuration.title) }
}
```

Access control keeps framework API small. The app needs the feature entry point, not every helper view inside the feature.

### How To Study This Example

- Identify which part is syntax and which part is design.
- Ask what invalid state the example prevents.
- Ask what would need to change if the feature became async, generic, public, or shared across modules.
- Rewrite the example with one deliberate bug, then explain how a reviewer should catch it.
- Turn the example into a two-minute interview explanation with tradeoffs.

