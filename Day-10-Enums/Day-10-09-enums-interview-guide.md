# Day 10: Enums Interview Guide

## One-Minute Interview Answer

Enums model a finite set of states. Swift enums are powerful because they support associated values, raw values, methods, computed properties, protocol conformance, recursive cases, `CaseIterable`, and pattern matching. I use enums to replace stringly typed state, model screen states, define routes and actions, and make invalid state combinations impossible. In SwiftUI, enum-driven UI works well because `switch` rendering is exhaustive.

## Modern Swift 6.x Notes

Swift 6.3 added `@c` support that can expose Swift enums to C for interop scenarios. For iOS interviews, the bigger everyday point is still exhaustive switching and enum-driven state modeling. Swift 6 concurrency also benefits from immutable enum state snapshots.

## Junior Questions

What is an enum?

A type with a fixed set of cases.

What are associated values?

Extra data attached to a specific case.

## Senior Questions

Why are enums powerful in app architecture?

They make finite states explicit and prevent invalid combinations.

Raw value or associated value?

Raw values are fixed constants; associated values carry dynamic case-specific data.

## Common Traps

- Using strings for states
- Adding `default` and hiding future enum cases
- Using many booleans instead of one enum state
- Confusing raw values and associated values

## Topic-By-Topic Deep Dive

### Basic Enums

```swift
enum AuthState {
    case loggedOut
    case loggingIn
    case loggedIn
}
```

Enums replace fragile strings and make state finite.

### Associated Values

```swift
enum AuthState {
    case loggedOut
    case loggedIn(User)
    case failed(String)
}
```

Each case carries only the data it needs.

### Raw Values

```swift
enum HTTPMethod: String {
    case get = "GET"
    case post = "POST"
}
```

Use raw values for persistence, API constants, and simple interop.

### Recursive Enums

```swift
indirect enum MenuItem {
    case item(String)
    case group(String, [MenuItem])
}
```

Great for trees, menus, expressions, and nested structures.

### CaseIterable

```swift
enum Filter: CaseIterable {
    case all
    case favorites
}
```

Useful for static UI choices.

### Identifiable Enums

```swift
enum Sheet: Identifiable {
    case settings
    case profile(String)

    var id: String {
        switch self {
        case .settings: return "settings"
        case .profile(let id): return "profile-\(id)"
        }
    }
}
```

Great for SwiftUI sheets and routes.

### State Modeling In iOS

Avoid:

```swift
var isLoading = false
var user: User?
var error: String?
```

Prefer:

```swift
enum ProfileState {
    case loading
    case loaded(User)
    case failed(String)
}
```

### Enum-Driven UI

```swift
switch state {
case .loading:
    ProgressView()
case .loaded(let user):
    ProfileView(user: user)
case .failed(let message):
    Text(message)
}
```

Senior answer:

Enums make UI rendering exhaustive and prevent invalid combinations.

## Production Decision Table

| Need | Enum Feature |
| --- | --- |
| Fixed set of choices | Basic enum |
| State with data | Associated values |
| API/persistence value | Raw values |
| Tree-like structure | Recursive enum |
| All static choices | `CaseIterable` |
| SwiftUI identity | `Identifiable` |
| Screen state | Enum state machine |

## Final Revision

- Basic enum: fixed cases
- Associated values: case-specific data
- Raw values: fixed backing values
- Recursive enums need `indirect`
- `CaseIterable` gives `allCases`
- `Identifiable` helps SwiftUI
- Enums are ideal for state modeling
