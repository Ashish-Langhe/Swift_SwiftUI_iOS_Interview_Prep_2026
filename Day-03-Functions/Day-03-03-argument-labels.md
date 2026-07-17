# Day 3: Argument Labels

## Core Idea

Swift functions can have argument labels that make calls read like sentences.

```swift
func greet(person name: String) {
    print("Hello, \(name)")
}

greet(person: "Riya")
```

Here:

- `person` is the external argument label
- `name` is the internal parameter name

## Default Rule

By default, the first parameter has no external label for functions, and later parameters use their parameter names as labels.

```swift
func move(from start: String, to end: String) {
    print("Move from \(start) to \(end)")
}

move(from: "Home", to: "Settings")
```

For many functions, Swift API design prefers calls that read naturally.

## Omitting Labels With Underscore

Use `_` to omit an external label.

```swift
func add(_ first: Int, _ second: Int) -> Int {
    first + second
}

let result = add(10, 20)
```

This is good for math-like functions where labels would add noise.

## External And Internal Names

```swift
func scheduleMeeting(on date: Date, with attendee: String) {
    print("Meeting with \(attendee) on \(date)")
}
```

Call site:

```swift
scheduleMeeting(on: Date(), with: "Aarav")
```

Inside the function:

```swift
date
attendee
```

## Why Labels Matter

Compare:

```swift
func resize(_ width: Int, _ height: Int) { }
resize(100, 200)
```

Better:

```swift
func resize(width: Int, height: Int) { }
resize(width: 100, height: 200)
```

Labels reduce mistakes when multiple parameters have the same type.

## Real iOS Use Cases

### Navigation

```swift
func navigate(to route: String, animated: Bool) {
    print("Navigate to \(route), animated: \(animated)")
}

navigate(to: "Profile", animated: true)
```

### Analytics

```swift
func track(event name: String, properties: [String: String]) {
    print("Track \(name): \(properties)")
}

track(event: "button_tap", properties: ["id": "pay_now"])
```

### Networking

```swift
func fetchUser(withId id: String) {
    print("Fetch user \(id)")
}

fetchUser(withId: "user-101")
```

## Junior-Level Interview Answer

Argument labels are names used when calling a function. They make the call easier to read.

```swift
func greet(name: String) { }
greet(name: "Meera")
```

## Mid-Level Interview Answer

Swift uses argument labels to improve API readability. External labels describe the role of the argument at the call site, while internal names are used inside the function body.

## Senior-Level Interview Answer

Argument labels are part of API design. A good Swift function should read clearly at the call site. Labels are especially important when parameters share the same type, because they prevent accidental ordering bugs. I omit labels only when the function is naturally positional, such as math operations or very common transformations.

## Common Mistakes

### Removing Labels Too Often

```swift
func createUser(_ name: String, _ email: String, _ city: String) { }
createUser("Riya", "riya@example.com", "Pune")
```

Better:

```swift
func createUser(name: String, email: String, city: String) { }
createUser(name: "Riya", email: "riya@example.com", city: "Pune")
```

### Label Repetition

Avoid awkward repetition:

```swift
func fetchUser(userId: String) { }
```

Better:

```swift
func fetchUser(withId id: String) { }
```

## Quick Interview Notes

- Argument labels appear at the call site.
- Parameter names are used inside the function.
- `_` removes the external label.
- Labels improve readability and reduce ordering mistakes.
- Good Swift APIs read naturally.

## Practice Questions

1. What is an argument label?
2. What does `_` do in a parameter list?
3. Why are labels useful when parameters have the same type?
4. What is the difference between external and internal names?
5. Give an example of a well-labeled Swift function.

