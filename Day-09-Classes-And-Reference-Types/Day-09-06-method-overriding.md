# Day 9: Method Overriding

## Core Idea

Subclasses can override superclass methods.

```swift
class BaseViewController {
    func setup() { }
}

class ProfileViewController: BaseViewController {
    override func setup() {
        super.setup()
        print("Profile setup")
    }
}
```

## Real iOS Use Cases

- `viewDidLoad`
- `viewWillAppear`
- Custom drawing
- UIKit lifecycle methods

## Interview Levels

Junior:

Override changes superclass behavior.

Senior:

Overriding is powerful but can hide control flow. Use clear superclass contracts and call `super` when required by framework behavior.

## Quick Notes

- Use `override`
- `super` calls parent implementation
- Common in UIKit
- Avoid deep inheritance chains

## Interview Depth

Junior answer: Overriding means a subclass provides its own version of a superclass method.

Mid-level answer: Swift requires the `override` keyword so overriding is explicit. `super` calls the parent implementation.

Senior answer: Overriding is part of a contract. Some framework methods require calling `super`; some do not. Know the lifecycle and avoid hidden behavior spread across many subclasses.

iOS use case:

```swift
override func viewWillAppear(_ animated: Bool) {
    super.viewWillAppear(animated)
    reloadData()
}
```

Common mistakes:

- Forgetting `super`.
- Overriding too many lifecycle methods.
- Creating fragile superclass hooks.

Practice:

1. Override `viewDidLoad`.
2. Explain `super`.
3. Explain risk of deep override chains.
