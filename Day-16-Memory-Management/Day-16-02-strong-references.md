# Day 16: Strong References

## Core Idea

A strong reference keeps a class instance alive.

By default, references are strong.

```swift
final class User { }

var user: User? = User()
```

The `user` variable strongly references the `User` instance.

## Ownership Rule

Strong should mean "I own this object" or "I need this object to stay alive."

```swift
final class CheckoutViewModel {
    private let paymentService: PaymentService

    init(paymentService: PaymentService) {
        self.paymentService = paymentService
    }
}
```

The view model strongly owns the service because it needs the service while the view model is alive.

## Multiple Strong References

```swift
final class Session {
    deinit {
        print("Session deallocated")
    }
}

var first: Session? = Session()
var second = first

first = nil
print("Still alive")

second = nil
print("Now can deallocate")
```

The object stays alive until all strong references are gone.

## Strong Reference Graphs

Healthy graph:

```text
ViewController -> ViewModel -> Service
```

Problem graph:

```text
ViewController -> ViewModel -> ViewController
```

If every arrow is strong, objects may never deallocate.

## Strong Collections

Collections strongly hold class instances.

```swift
var coordinators: [Coordinator] = []
coordinators.append(childCoordinator)
```

The array keeps `childCoordinator` alive. This is common in coordinator architectures, but you must remove child coordinators when flows finish.

```swift
func childDidFinish(_ child: Coordinator) {
    coordinators.removeAll { $0 === child }
}
```

## Strong Singletons And Caches

Singletons and caches can accidentally keep objects alive forever.

```swift
final class ImageCache {
    static let shared = ImageCache()
    private var images: [URL: UIImage] = [:]
}
```

This can be correct, but a cache needs eviction rules. Otherwise, it becomes a memory growth source.

## Ownership Meaning

A strong reference usually means ownership.

```swift
final class ProfileViewModel {
    private let service: ProfileService

    init(service: ProfileService) {
        self.service = service
    }
}
```

The view model strongly owns the service dependency.

## Real iOS Use Cases

Strong references are correct for:

- A view model owning a service
- A coordinator owning child coordinators
- A cache owning cached values
- A view controller owning its view model

```swift
final class ProfileViewController: UIViewController {
    private let viewModel: ProfileViewModel

    init(viewModel: ProfileViewModel) {
        self.viewModel = viewModel
        super.init(nibName: nil, bundle: nil)
    }
}
```

## When Strong References Become Dangerous

Strong references become dangerous when two objects own each other.

```swift
final class Parent {
    var child: Child?
}

final class Child {
    var parent: Parent?
}
```

If both are strong, neither object can be released.

## Advanced Senior Reasoning

When reviewing memory ownership, ask:

1. Who creates this object?
2. Who owns it?
3. Who only observes it?
4. Who calls back to whom?
5. Who should be released first?
6. Is there any path from the object back to itself through strong references?

If there is a strong path back to itself, you likely have a cycle.

## Junior-Level Interview Answer

A strong reference keeps an object alive. References are strong by default.

## Mid-Level Interview Answer

Strong references represent ownership. ARC deallocates an object only after all strong references are gone.

## Senior-Level Interview Answer

Strong references should express ownership. In architecture, I ask: who owns this object, and who merely observes or calls back? Strong ownership is correct for dependencies and children, but incorrect for delegates, back-references, and many callbacks. Clear ownership prevents leaks.

## Common Mistakes

- Strong delegate references.
- Parent and child strongly referencing each other.
- Storing closures strongly while closures capture `self`.
- Long-lived singletons strongly holding short-lived screens.

## Interview Progression

Basic:

Strong references keep objects alive.

Intermediate:

Strong references represent ownership. An object is released only when all strong references disappear.

Advanced:

Strong ownership should usually form a directed ownership graph. Back-references, delegates, callbacks, and child-to-parent relationships usually should not be strong. Long-lived owners like singletons and caches need explicit lifetime and eviction policy.

## Practice

1. Draw ownership for view controller, view model, service.
2. Identify which references should be strong.
3. Fix a parent-child retain cycle.
