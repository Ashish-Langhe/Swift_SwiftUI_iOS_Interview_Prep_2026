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
