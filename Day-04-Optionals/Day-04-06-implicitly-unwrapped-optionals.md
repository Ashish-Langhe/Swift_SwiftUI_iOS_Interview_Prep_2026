# Day 4: Implicitly Unwrapped Optionals

## Core Idea

An implicitly unwrapped optional is written with `!`.

```swift
var titleLabel: UILabel!
```

It is still optional, but Swift lets you use it as if it were non-optional.

If it is nil when accessed, the app crashes.

## Why IUOs Exist

Implicitly unwrapped optionals are useful when a value starts as nil but is guaranteed to be set before use.

Classic UIKit example:

```swift
@IBOutlet weak var titleLabel: UILabel!
```

The outlet is nil before the view is loaded, then UIKit connects it from the storyboard or xib.

## IUO Vs Normal Optional

Normal optional:

```swift
var name: String?
print(name?.count)
```

Implicitly unwrapped optional:

```swift
var name: String!
print(name.count)
```

If `name` is nil, the second version crashes.

## Modern Swift Guidance

Use IUOs sparingly.

Prefer normal optionals unless there is a strong lifecycle guarantee.

```swift
var selectedUser: User?
```

Avoid:

```swift
var selectedUser: User!
```

Unless you can truly guarantee it is set before use.

## Real iOS Use Cases

### UIKit IBOutlet

```swift
import UIKit

final class ProfileViewController: UIViewController {
    @IBOutlet private weak var nameLabel: UILabel!

    override func viewDidLoad() {
        super.viewDidLoad()
        nameLabel.text = "Profile"
    }
}
```

This is common because UIKit connects outlets after initialization.

### Test Setup

```swift
final class LoginViewModelTests {
    var viewModel: LoginViewModel!

    func setUp() {
        viewModel = LoginViewModel()
    }
}
```

Some test suites use IUOs for values initialized in setup.

## Safer Alternatives

### Initialize In Constructor

```swift
final class ProfileViewModel {
    let user: User

    init(user: User) {
        self.user = user
    }
}
```

### Use Normal Optional

```swift
var selectedUser: User?
```

### Use Lazy Property

```swift
lazy var formatter: DateFormatter = {
    let formatter = DateFormatter()
    formatter.dateStyle = .medium
    return formatter
}()
```

## Junior-Level Interview Answer

An implicitly unwrapped optional is an optional that can be used without unwrapping, but it crashes if nil.

## Mid-Level Interview Answer

IUOs are useful when a value cannot be set during initialization but is guaranteed before use, such as UIKit outlets. They should be avoided for normal app state.

## Senior-Level Interview Answer

IUOs are a lifecycle escape hatch. They encode "this starts nil, but should be non-nil before use." That is a risky contract, so I limit IUOs to framework-driven lifecycle cases like IBOutlets or controlled test setup. For domain state, I prefer constructor injection, normal optionals, lazy properties, or explicit state enums.

## Quick Interview Notes

- IUO syntax is `Type!`.
- IUOs are still optional.
- Accessing nil IUO crashes.
- Common with UIKit `@IBOutlet`.
- Avoid IUOs for ordinary model or view state.

## Practice Questions

1. What is an implicitly unwrapped optional?
2. How is `String!` different from `String?`?
3. Why are IBOutlets often IUOs?
4. What happens if an IUO is nil when accessed?
5. What are safer alternatives to IUOs?

