# Day 2: Loops - For-In, While, And Repeat-While

## What You Will Learn

- How to repeat work using loops
- Difference between `for-in`, `while`, and `repeat-while`
- Loop control using `break` and `continue`
- Iterating arrays, dictionaries, sets, strings, and ranges
- Performance and readability considerations
- Junior to senior-level interview explanations

## Core Idea

Loops let you run the same block of code multiple times.

Swift has three common loop forms:

- `for-in`
- `while`
- `repeat-while`

## For-In Loop

Use `for-in` when you want to iterate over a sequence or collection.

```swift
let names = ["Aarav", "Meera", "Kabir"]

for name in names {
    print(name)
}
```

Use it with ranges:

```swift
for number in 1...5 {
    print(number)
}
```

Output:

```text
1
2
3
4
5
```

## Half-Open Ranges

```swift
for index in 0..<5 {
    print(index)
}
```

Output:

```text
0
1
2
3
4
```

Half-open ranges are useful for zero-based indexing.

## Iterating Arrays With Index

Prefer `enumerated()` when you need both index and value.

```swift
let topics = ["Variables", "Control Flow", "Functions"]

for (index, topic) in topics.enumerated() {
    print("\(index + 1). \(topic)")
}
```

Avoid unnecessary index-based loops:

```swift
for index in 0..<topics.count {
    print(topics[index])
}
```

This is valid but less expressive when you only need values.

## Iterating Dictionaries

```swift
let scores = [
    "Aarav": 90,
    "Meera": 85,
    "Kabir": 92
]

for (name, score) in scores {
    print("\(name): \(score)")
}
```

Dictionary order is not something your business logic should depend on unless you explicitly sort.

```swift
for (name, score) in scores.sorted(by: { $0.key < $1.key }) {
    print("\(name): \(score)")
}
```

## Iterating Sets

```swift
let uniqueSkills: Set<String> = ["Swift", "SwiftUI", "UIKit"]

for skill in uniqueSkills {
    print(skill)
}
```

Set order is not guaranteed. Sort if order matters.

```swift
for skill in uniqueSkills.sorted() {
    print(skill)
}
```

## Iterating Strings

```swift
let word = "Swift"

for character in word {
    print(character)
}
```

Swift strings are Unicode-aware. A `Character` may be more complex than one byte.

## Where In For-In

Use `where` to filter inside a loop.

```swift
let numbers = [1, 2, 3, 4, 5, 6]

for number in numbers where number.isMultiple(of: 2) {
    print(number)
}
```

This prints only even numbers.

## While Loop

Use `while` when the number of iterations is not known in advance.

```swift
var attempts = 0

while attempts < 3 {
    print("Attempt \(attempts + 1)")
    attempts += 1
}
```

The condition is checked before each iteration.

If the condition is false at the start, the loop body never runs.

## Repeat-While Loop

`repeat-while` runs at least once.

```swift
var countdown = 3

repeat {
    print(countdown)
    countdown -= 1
} while countdown > 0
```

Use this when the action must happen before checking the condition.

## Break

`break` exits the loop immediately.

```swift
let numbers = [1, 3, 5, 8, 9]

for number in numbers {
    if number.isMultiple(of: 2) {
        print("Found even number: \(number)")
        break
    }
}
```

## Continue

`continue` skips the current iteration and moves to the next one.

```swift
let numbers = [1, 2, 3, 4, 5]

for number in numbers {
    if number.isMultiple(of: 2) {
        continue
    }

    print(number)
}
```

This prints only odd numbers.

## Labeled Statements

Labels help break out of nested loops.

```swift
outerLoop: for row in 1...3 {
    for column in 1...3 {
        if row == 2 && column == 2 {
            break outerLoop
        }

        print("row: \(row), column: \(column)")
    }
}
```

Use labels sparingly. Often, extracting logic into a function is cleaner.

## Infinite Loops

```swift
while true {
    print("Running")
    break
}
```

Infinite loops are sometimes used in systems code or event loops, but most app code should avoid them unless there is a clear exit path.

## Real iOS Use Cases

### Rendering Rows

```swift
let transactions = ["Food", "Travel", "Shopping"]

for transaction in transactions {
    print("Render row for \(transaction)")
}
```

### Retry Logic

```swift
var retryCount = 0
let maxRetries = 3

while retryCount < maxRetries {
    print("Trying request")
    retryCount += 1
}
```

### Validating Form Fields

```swift
let fields = ["email", "", "password"]

for field in fields {
    if field.isEmpty {
        print("Invalid field found")
        break
    }
}
```

### SwiftUI List

In SwiftUI, you usually do not write a manual loop inside `body`; you use `ForEach`.

```swift
import SwiftUI

struct TopicListView: View {
    let topics = ["Variables", "Control Flow", "Functions"]

    var body: some View {
        List(topics, id: \.self) { topic in
            Text(topic)
        }
    }
}
```

`ForEach` is SwiftUI's declarative way to describe repeated views.

## Junior-Level Interview Answer

Question: What are loops in Swift?

Answer:

Loops repeat a block of code. `for-in` is used to iterate collections or ranges. `while` runs while a condition is true. `repeat-while` runs at least once and then checks the condition.

## Mid-Level Interview Answer

Question: When do you choose `for-in` vs `while`?

Answer:

I use `for-in` when iterating over a known sequence, such as an array, dictionary, set, or range. I use `while` when the number of iterations is not known ahead of time and depends on a condition, such as retrying until success or reading until no data remains.

## Senior-Level Interview Answer

Question: What loop choices matter in production Swift code?

Answer:

I prefer the loop form that communicates intent. `for-in` is ideal for collections because it avoids manual indexing errors. I avoid relying on unordered collection iteration order. In performance-sensitive code, I consider allocation, sorting cost, early exits, and whether a higher-order method like `first(where:)`, `contains(where:)`, or `map` expresses the operation more clearly. In SwiftUI, I use `ForEach` for repeated views because identity matters for diffing and rendering.

## Quick Interview Notes

- `for-in` iterates over sequences.
- `while` checks the condition before running.
- `repeat-while` runs at least once.
- `break` exits a loop.
- `continue` skips to the next iteration.
- Use `enumerated()` for index and value.
- Do not depend on dictionary or set order unless sorted.
- Use SwiftUI `ForEach` for repeated views.

## Points To Remember

- Prefer direct iteration over manual indexing.
- Choose loop type based on intent.
- Avoid infinite loops without a clear exit.
- Use `where` in loops for simple filtering.
- For complex nested loops, consider extracting a function.

## Practice Questions

1. What is the difference between `while` and `repeat-while`?
2. How do you get both index and value from an array?
3. What does `continue` do?
4. Why should you avoid depending on dictionary order?
5. How is SwiftUI `ForEach` different from a normal Swift `for` loop?

## Deep Dive Expansion

### Why This Topic Matters In Real Apps

Loops - For-In, While, And Repeat-While is not only a syntax topic. In production Swift, it affects branch clarity, exhaustiveness, early exits, and reducing nested logic. A junior developer should be able to recognize the syntax and explain the basic behavior. A mid-level developer should know when the feature improves clarity or safety. A senior developer should understand how it changes API design, testability, performance, or long-term maintenance.

In iOS work, this usually appears while validating form input, handling API states, and routing user actions. The important question is not "Can I write the syntax?" The important question is "Does this code make invalid states harder, make ownership clearer, and make future changes safer?"

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

Use this checklist when applying Loops - For-In, While, And Repeat-While in an app feature:

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

1. Write a minimal example that shows Loops - For-In, While, And Repeat-While correctly.
2. Write a second example that demonstrates a common mistake.
3. Refactor the mistake into a safer version.
4. Explain the difference between the beginner solution and the production-ready solution.
5. Add one test case or reasoning check that proves the behavior.

## Applied Day-Level Example

The following example is not the only way to use Loops - For-In, While, And Repeat-While, but it shows the kind of production shape you should connect this topic to:

```swift
enum CheckoutState {
    case emptyCart
    case needsAddress
    case readyToPay(total: Decimal)
    case processing
    case failed(String)
}

func primaryActionTitle(for state: CheckoutState) -> String {
    switch state {
    case .emptyCart:
        return "Continue Shopping"
    case .needsAddress:
        return "Add Address"
    case .readyToPay:
        return "Pay Now"
    case .processing:
        return "Processing"
    case .failed:
        return "Try Again"
    }
}
```

This is the production mindset for control flow: branch on domain state, keep each case intentional, and let the compiler force you to handle new states.

### How To Study This Example

- Identify which part is syntax and which part is design.
- Ask what invalid state the example prevents.
- Ask what would need to change if the feature became async, generic, public, or shared across modules.
- Rewrite the example with one deliberate bug, then explain how a reviewer should catch it.
- Turn the example into a two-minute interview explanation with tradeoffs.

