# Day 17: `open`

## What `open` Means

`open` is the least restrictive access level for classes and class members.

It allows code outside the module to:

- Use the class
- Subclass the class
- Override open methods and properties

```swift
open class BaseViewController: UIViewController {
    open func configure() {
        // Subclasses outside this module may override.
    }
}
```

`open` applies only to classes and class members. Structs and enums cannot be subclassed, so they do not use `open`.

## Beginner Mental Model

Use `open` when you want:

```text
External modules may inherit from this class or override this member.
```

Use `public` when external modules may use the type but should not customize it through inheritance.

## `public` vs `open`

```swift
public class PublicBase {
    public func doWork() { }
}
```

External modules can create and use `PublicBase`, but they cannot subclass it.

```swift
open class OpenBase {
    open func doWork() { }
}
```

External modules can subclass `OpenBase` and override `doWork`.

## Open Methods

A class can be open while some methods are not open.

```swift
open class AnalyticsScreen: UIViewController {
    public final func trackScreenView() {
        // Cannot be overridden.
    }

    open func screenName() -> String {
        "AnalyticsScreen"
    }
}
```

External subclasses can override `screenName`, but not `trackScreenView`.

## Real iOS Use Cases

Use `open` for:

- Framework base classes meant for subclassing
- SDK customization points
- UIKit/AppKit-style inheritance APIs
- Template method patterns
- Base view controllers in a reusable framework

```swift
open class FormFieldView: UIView {
    public private(set) var text: String = ""

    open func validate(_ text: String) -> Bool {
        true
    }

    public func update(text: String) {
        guard validate(text) else { return }
        self.text = text
    }
}
```

External modules can customize validation by subclassing.

## Open Is A Strong Promise

`open` is more than public.

When you mark a class open, you promise that external subclassing is supported.

That means you should think about:

- Which methods are safe to override
- Which state must remain protected
- Whether methods should be `final`
- Whether initialization is subclass-safe
- Whether documentation explains override expectations
- Whether future changes may break subclasses

## Template Method Pattern

```swift
open class BaseCoordinator {
    public final func start() {
        prepare()
        showInitialScreen()
    }

    open func prepare() {
        // Optional customization.
    }

    open func showInitialScreen() {
        fatalError("Subclasses must override")
    }
}
```

`start()` is final to preserve sequence.

Subclasses override customization points.

## Prefer Composition When Possible

Open inheritance can be powerful, but it also creates tight coupling.

Often composition is better:

```swift
public protocol FormFieldValidating {
    func validate(_ text: String) -> Bool
}

public final class FormFieldView: UIView {
    private let validator: FormFieldValidating

    public init(validator: FormFieldValidating) {
        self.validator = validator
        super.init(frame: .zero)
    }

    required init?(coder: NSCoder) {
        fatalError("init(coder:) has not been implemented")
    }
}
```

This exposes customization without inheritance.

## Open And `final`

`final` prevents subclassing or overriding.

```swift
public final class APIClient {
    public init() { }
}
```

This is often better for services because external subclassing can break assumptions.

Use `open` intentionally, not by default.

## Advanced Design Notes

Open classes are hard to evolve because external subclasses can depend on internal timing and behavior.

Senior checklist before `open`:

- Is inheritance the intended extension mechanism?
- Are override points documented?
- Can subclasses violate invariants?
- Should the type be `public final` instead?
- Would a protocol or closure-based customization be safer?
- Are initializers designed for subclassing?

## Modern Swift 6.x Notes

`open` remains stable in Swift 6.x.

Modern Swift design often favors:

- Value types
- Protocols
- Composition
- Actors for isolated reference state
- Final classes for services

This does not make `open` obsolete. It means `open` should be reserved for APIs where inheritance is truly part of the design.

## Junior Interview Answer

`open` allows a class to be subclassed and open members to be overridden from outside the module.

## Mid-Level Interview Answer

`public` allows external use, but not external subclassing or overriding. `open` allows external subclassing and overriding. Use `open` only when you intentionally support inheritance.

## Senior Interview Answer

`open` is a long-term framework design commitment. It exposes inheritance as an extension point, which makes future changes harder because external subclasses may rely on behavior. I prefer `public final`, protocols, or composition unless subclassing is explicitly part of the API.

## Points To Remember

- `open` is only for classes and class members.
- `open` is broader than `public`.
- Public classes cannot be subclassed outside the module.
- Open classes can be subclassed outside the module.
- Prefer `final` unless subclassing is needed.

## Practice

1. Explain why `UIViewController` is subclassable.
2. Design an open base class with one final method and one open customization point.
3. Rewrite an open inheritance design using a protocol.
4. Explain why open APIs are harder to maintain.
