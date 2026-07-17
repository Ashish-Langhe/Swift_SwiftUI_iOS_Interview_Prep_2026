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

## Deep Dive Expansion

### Why This Topic Matters In Real Apps

open is not only a syntax topic. In production Swift, it affects API surface, encapsulation, package boundaries, framework evolution, and source compatibility. A junior developer should be able to recognize the syntax and explain the basic behavior. A mid-level developer should know when the feature improves clarity or safety. A senior developer should understand how it changes API design, testability, performance, or long-term maintenance.

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

Use this checklist when applying open in an app feature:

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

1. Write a minimal example that shows open correctly.
2. Write a second example that demonstrates a common mistake.
3. Refactor the mistake into a safer version.
4. Explain the difference between the beginner solution and the production-ready solution.
5. Add one test case or reasoning check that proves the behavior.

## Applied Day-Level Example

The following example is not the only way to use open, but it shows the kind of production shape you should connect this topic to:

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

