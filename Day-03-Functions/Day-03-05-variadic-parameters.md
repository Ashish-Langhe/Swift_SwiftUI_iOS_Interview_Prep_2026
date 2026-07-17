# Day 3: Variadic Parameters

## Core Idea

A variadic parameter accepts zero or more values of the same type.

```swift
func sum(_ numbers: Int...) -> Int {
    var total = 0

    for number in numbers {
        total += number
    }

    return total
}

let result = sum(1, 2, 3, 4)
```

Inside the function, `numbers` behaves like an array.

## Syntax

```swift
func functionName(_ values: Type...) { }
```

Example:

```swift
func printMessages(_ messages: String...) {
    for message in messages {
        print(message)
    }
}

printMessages("One", "Two", "Three")
```

## Zero Arguments

Variadic parameters can receive no values.

```swift
printMessages()
```

The `messages` collection is empty.

## Real iOS Use Cases

### Logging

```swift
func log(_ items: Any...) {
    for item in items {
        print(item)
    }
}

log("User id", 101, "loaded")
```

### Validation

```swift
func allFieldsFilled(_ fields: String...) -> Bool {
    fields.allSatisfy { !$0.isEmpty }
}

let canSubmit = allFieldsFilled("email@example.com", "password")
```

### Combining Predicates

```swift
func allTrue(_ conditions: Bool...) -> Bool {
    conditions.allSatisfy { $0 }
}

let canAccess = allTrue(true, true, false)
```

## Variadic Parameters Vs Arrays

Variadic:

```swift
func average(_ numbers: Double...) -> Double {
    numbers.reduce(0, +) / Double(numbers.count)
}

average(10, 20, 30)
```

Array:

```swift
func average(_ numbers: [Double]) -> Double {
    numbers.reduce(0, +) / Double(numbers.count)
}

average([10, 20, 30])
```

Use variadic parameters when call-site convenience is valuable. Use arrays when the values already exist as a collection or the list may be large/dynamic.

## Common Mistake: Empty Variadic Edge Case

This crashes when no numbers are passed because of division by zero.

```swift
func average(_ numbers: Double...) -> Double {
    numbers.reduce(0, +) / Double(numbers.count)
}
```

Safer:

```swift
func average(_ numbers: Double...) -> Double? {
    guard !numbers.isEmpty else {
        return nil
    }

    return numbers.reduce(0, +) / Double(numbers.count)
}
```

## Junior-Level Interview Answer

A variadic parameter allows passing multiple values to a single parameter.

```swift
func printNames(_ names: String...) { }
printNames("A", "B", "C")
```

## Mid-Level Interview Answer

Inside the function, a variadic parameter behaves like an array. It is useful for convenience APIs such as logging, formatting, or validation.

## Senior-Level Interview Answer

Variadic parameters are call-site ergonomics. I use them when the number of arguments is naturally small and manually listed. If values are dynamic, already collected, or potentially large, an array is a better API. I also handle the zero-argument case explicitly when the function requires at least one value.

## Quick Interview Notes

- Variadic parameters use `...`.
- They accept zero or more values.
- Inside the function, they behave like arrays.
- Use arrays for dynamic collections.
- Handle empty input carefully.

## Practice Questions

1. What is a variadic parameter?
2. How does a variadic parameter behave inside a function?
3. Can a variadic parameter receive zero values?
4. When is an array better than a variadic parameter?
5. What bug can happen in an average function with empty input?

## Deep Dive Expansion

### Why This Topic Matters In Real Apps

Variadic Parameters is not only a syntax topic. In production Swift, it affects clear API design, parameter labels, ownership of side effects, and composability. A junior developer should be able to recognize the syntax and explain the basic behavior. A mid-level developer should know when the feature improves clarity or safety. A senior developer should understand how it changes API design, testability, performance, or long-term maintenance.

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

Use this checklist when applying Variadic Parameters in an app feature:

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

1. Write a minimal example that shows Variadic Parameters correctly.
2. Write a second example that demonstrates a common mistake.
3. Refactor the mistake into a safer version.
4. Explain the difference between the beginner solution and the production-ready solution.
5. Add one test case or reasoning check that proves the behavior.

## Applied Day-Level Example

The following example is not the only way to use Variadic Parameters, but it shows the kind of production shape you should connect this topic to:

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

