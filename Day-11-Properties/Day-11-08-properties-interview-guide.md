# Day 11: Properties Interview Guide

## One-Minute Interview Answer

Swift properties store or compute values on types. Stored properties hold state, computed properties derive state, lazy properties delay initialization, observers respond to changes, type properties belong to the type, and access control protects invariants. Property wrappers add reusable behavior around properties and are heavily used in SwiftUI. In modern Swift, property mutability matters for concurrency safety, so I prefer immutable state and controlled mutation.

## Modern Swift 6.x Notes

Swift 6 concurrency checking makes property ownership important. Mutable shared properties should be isolated. `static let` is safe for constants, while mutable static state should be avoided or protected. Swift 6.2 Observation updates also make property changes more relevant for UI state observation.

## Junior Questions

What is a stored property? A property that stores a value.

What is a computed property? A property that calculates a value.

What is `lazy`? Delayed initialization until first use.

## Senior Questions

When use `private(set)`? When external code can read state but only the owner should mutate it.

When avoid property observers? When they hide heavy side effects or make state flow hard to trace.

## Common Traps

- Doing heavy work in computed properties.
- Using mutable static properties casually.
- Overusing property wrappers.
- Exposing public mutable state.

## Topic-By-Topic Deep Dive

### Stored Properties

Stored properties are the actual state of a type. In interviews, do not stop at "they store values." Explain that they define ownership and mutability.

```swift
struct BankAccount {
    let id: String
    private(set) var balance: Decimal
}
```

Here, `id` is immutable identity data. `balance` can change, but only inside the type because external code should not freely mutate account balance.

Junior answer:

Stored properties hold values inside a type.

Senior answer:

Stored properties define the state boundary of a type. Their mutability should match domain rules. In Swift 6-style concurrent code, immutable stored properties are easier to share safely, while mutable state needs clear ownership or actor isolation.

### Computed Properties

Computed properties are derived values.

```swift
struct Cart {
    let items: [CartItem]

    var total: Decimal {
        items.reduce(0) { $0 + $1.price }
    }
}
```

This is readable if `items` is small. If `items` is huge and `total` is accessed many times, a computed property may hide expensive work.

Senior rule:

Use computed properties for cheap, deterministic derivations. Use methods for work that is expensive, asynchronous, throwing, or surprising.

### Lazy Properties

Lazy properties are useful when setup is expensive or depends on `self`.

```swift
final class ReportGenerator {
    lazy var formatter: DateFormatter = {
        let formatter = DateFormatter()
        formatter.dateStyle = .long
        return formatter
    }()
}
```

Senior caution:

Lazy initialization timing is hidden. If multiple tasks access a lazy property on shared state, do not treat `lazy` as a concurrency primitive.

### Property Observers

Property observers help react to state changes.

```swift
var query = "" {
    didSet {
        guard query != oldValue else { return }
        print("Search query changed")
    }
}
```

Senior caution:

Observers can make side effects invisible. Avoid network calls, heavy parsing, or navigation inside observers. Prefer explicit methods for important workflows.

### Type Properties

Type properties are shared by all instances.

```swift
struct API {
    static let baseURL = URL(string: "https://api.example.com")!
}
```

`static let` constants are clean. `static var` mutable state is global state and can be dangerous.

### Access Control On Properties

Access control protects invariants.

```swift
final class SessionStore {
    private(set) var currentUser: User?

    func update(user: User?) {
        currentUser = user
    }
}
```

This lets the rest of the app observe state but prevents random mutation.

### Property Wrappers

Property wrappers move repeated property behavior into reusable types.

SwiftUI examples:

```swift
@State private var count = 0
@Binding var isPresented: Bool
@Environment(\.dismiss) private var dismiss
```

Senior answer:

Property wrappers are powerful because they change how access is mediated. But that also means they can hide behavior. In interviews, mention that wrappers should improve clarity, not make property reads/writes mysterious.

## Production Decision Table

| Need | Property Tool |
| --- | --- |
| Store model state | Stored property |
| Derive cheap display value | Computed property |
| Create expensive helper only when needed | Lazy property |
| React to simple local change | Property observer |
| Shared constant | `static let` |
| Readable outside, writable inside | `private(set)` |
| Reusable access behavior | Property wrapper |

## Final Revision

- Stored stores.
- Computed derives.
- Lazy delays.
- Observers react.
- Type properties belong to type.
- Access control protects invariants.
- Wrappers reuse property behavior.

## Deep Dive Expansion

### Why This Topic Matters In Real Apps

Properties Interview Guide is not only a syntax topic. In production Swift, it affects state ownership, computed behavior, lazy initialization, observation, and wrapper semantics. A junior developer should be able to recognize the syntax and explain the basic behavior. A mid-level developer should know when the feature improves clarity or safety. A senior developer should understand how it changes API design, testability, performance, or long-term maintenance.

In iOS work, this usually appears while view models, cached values, validation state, and SwiftUI data flow. The important question is not "Can I write the syntax?" The important question is "Does this code make invalid states harder, make ownership clearer, and make future changes safer?"

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

Use this checklist when applying Properties Interview Guide in an app feature:

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

- Turn each answer into a story: situation, decision, tradeoff, result, and verification.
- For senior interviews, avoid only definitions. Explain why the design prevents bugs in a real app.

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

1. Write a minimal example that shows Properties Interview Guide correctly.
2. Write a second example that demonstrates a common mistake.
3. Refactor the mistake into a safer version.
4. Explain the difference between the beginner solution and the production-ready solution.
5. Turn one answer into a real project story with tradeoffs and verification.

## Applied Day-Level Example

The following example is not the only way to use Properties Interview Guide, but it shows the kind of production shape you should connect this topic to:

```swift
final class CartViewModel: ObservableObject {
    @Published private(set) var items: [CartItem] = []

    var total: Decimal {
        items.reduce(0) { $0 + $1.price }
    }

    func add(_ item: CartItem) {
        items.append(item)
    }
}
```

Properties are where state ownership becomes visible. Stored properties hold truth, computed properties derive truth, and restricted setters protect mutation rules.

### How To Study This Example

- Identify which part is syntax and which part is design.
- Ask what invalid state the example prevents.
- Ask what would need to change if the feature became async, generic, public, or shared across modules.
- Rewrite the example with one deliberate bug, then explain how a reviewer should catch it.
- Turn the example into a two-minute interview explanation with tradeoffs.

