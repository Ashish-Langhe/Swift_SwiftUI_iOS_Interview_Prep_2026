# Day 11: Type Properties

## Core Idea

Type properties belong to the type itself, not an instance.

```swift
struct AppConfig {
    static let appName = "Interview Prep"
}

print(AppConfig.appName)
```

## Static Vs Class

`static` cannot be overridden.

`class` properties in classes can be overridden when written as computed properties.

```swift
class Theme {
    class var name: String { "Default" }
}
```

## Real iOS Use Cases

- Constants
- Shared configuration
- Factory values
- Notification names

## Modern Swift 6.x Notes

Mutable static properties are shared global state. In Swift concurrency, mutable shared state must be protected. Prefer immutable `static let` or actor-isolated storage.

## Interview Levels

Junior: Type properties are accessed on the type, not an object.

Senior: `static let` is excellent for constants. Mutable static state is risky because it is global shared state and can become a data-race source.

## Quick Notes

- Use `static`.
- Access with `Type.property`.
- Prefer `static let`.
- Avoid mutable global state.

## Interview Depth

Junior answer: Type properties belong to the type, not a specific instance.

Mid-level answer: Use `static let` for constants and shared configuration.

Senior answer: Mutable type properties are global shared state. In concurrent Swift, they can become data-race risks unless protected.

iOS use case:

```swift
struct APIEnvironment {
    static let productionBaseURL = URL(string: "https://api.example.com")!
}
```

Common mistakes: mutable static caches, test pollution, hidden global dependencies.

Practice: create static constant, explain static var risk, compare instance vs type property.
