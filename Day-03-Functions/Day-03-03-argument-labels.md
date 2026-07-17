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

## Deep Dive Expansion

### Why This Topic Matters In Real Apps

Argument Labels is not only a syntax topic. In production Swift, it affects clear API design, parameter labels, ownership of side effects, and composability. A junior developer should be able to recognize the syntax and explain the basic behavior. A mid-level developer should know when the feature improves clarity or safety. A senior developer should understand how it changes API design, testability, performance, or long-term maintenance.

In iOS work, this usually appears while building service methods, validation helpers, and reusable transformation functions. The important question is not "Can I write the syntax?" The important question is "Does this code make invalid states harder, make ownership clearer, and make future changes safer?"

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

Use this checklist when applying Argument Labels in an app feature:

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

- Prefer code that communicates intent at the call site, not only code that compiles.
- When a feature grows, revisit whether the type still owns the right responsibilities.

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

1. Write a minimal example that shows Argument Labels correctly.
2. Write a second example that demonstrates a common mistake.
3. Refactor the mistake into a safer version.
4. Explain the difference between the beginner solution and the production-ready solution.
5. Add one test case or reasoning check that proves the behavior.

## Applied Day-Level Example

The following example is not the only way to use Argument Labels, but it shows the kind of production shape you should connect this topic to:

```swift
struct PasswordRule {
    let message: String
    let validate: (String) -> Bool
}

func validatePassword(_ password: String, rules: [PasswordRule]) -> [String] {
    rules.compactMap { rule in
        rule.validate(password) ? nil : rule.message
    }
}

let rules = [
    PasswordRule(message: "Use at least 8 characters") { $0.count >= 8 },
    PasswordRule(message: "Add one number") { $0.contains(where: \.isNumber) }
]
```

This connects function design to reuse. The validation function has clear inputs and output, while each rule is injectable and easy to test.

### How To Study This Example

- Identify which part is syntax and which part is design.
- Ask what invalid state the example prevents.
- Ask what would need to change if the feature became async, generic, public, or shared across modules.
- Rewrite the example with one deliberate bug, then explain how a reviewer should catch it.
- Turn the example into a two-minute interview explanation with tradeoffs.

