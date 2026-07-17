# Day 3: Default Parameters

## Core Idea

Default parameters let a function provide default values for arguments.

```swift
func greet(name: String, punctuation: String = "!") {
    print("Hello, \(name)\(punctuation)")
}

greet(name: "Aarav")
greet(name: "Aarav", punctuation: ".")
```

If the caller does not pass `punctuation`, Swift uses `"!"`.

## Why Default Parameters Matter

They reduce overloads and keep common calls simple.

Without default parameter:

```swift
func fetchUsers(page: Int, pageSize: Int) { }
func fetchUsers(page: Int) {
    fetchUsers(page: page, pageSize: 20)
}
```

With default parameter:

```swift
func fetchUsers(page: Int, pageSize: Int = 20) { }
```

## Real iOS Use Cases

### Animation

```swift
func showToast(message: String, duration: TimeInterval = 2.0) {
    print("Show \(message) for \(duration) seconds")
}
```

### Networking

```swift
func request(
    path: String,
    method: String = "GET",
    timeout: TimeInterval = 30
) {
    print("\(method) \(path), timeout: \(timeout)")
}
```

### Analytics

```swift
func track(event name: String, properties: [String: String] = [:]) {
    print("Event: \(name), properties: \(properties)")
}
```

## Good Defaults

Good defaults should represent the common case.

```swift
func loadImage(from url: URL, useCache: Bool = true) { }
```

The common behavior is usually to use cache.

## Bad Defaults

Avoid defaults that hide important decisions.

```swift
func deleteUser(id: String, confirm: Bool = true) { }
```

Deletion is important enough that confirmation should probably be explicit.

## Default Parameters And API Design

Default parameters are useful, but too many can make a function unclear.

```swift
func createButton(
    title: String,
    color: String = "blue",
    isEnabled: Bool = true,
    showsIcon: Bool = false,
    cornerRadius: Double = 8
) { }
```

This may be fine for small utilities, but if configuration grows, consider a configuration struct.

```swift
struct ButtonConfiguration {
    var color: String = "blue"
    var isEnabled: Bool = true
    var showsIcon: Bool = false
    var cornerRadius: Double = 8
}
```

## Junior-Level Interview Answer

Default parameters are values used when the caller does not provide an argument.

## Mid-Level Interview Answer

Default parameters simplify common function calls and reduce the need for multiple overloads. They should represent sensible defaults.

## Senior-Level Interview Answer

Default parameters are API ergonomics. They make common cases concise, but they can hide important behavior if overused. For critical actions, security-sensitive operations, or domain decisions, I prefer explicit arguments. If a function has many defaults, a configuration type may be clearer and easier to evolve.

## Quick Interview Notes

- Default values are declared in the parameter list.
- Callers can omit arguments with defaults.
- Defaults should represent common safe behavior.
- Too many defaults can signal a configuration object is needed.
- Avoid defaults for dangerous or business-critical behavior.

## Practice Questions

1. What is a default parameter?
2. Why are default parameters useful?
3. When can default parameters be harmful?
4. How can default parameters reduce overloads?
5. When should you use a configuration struct instead?

