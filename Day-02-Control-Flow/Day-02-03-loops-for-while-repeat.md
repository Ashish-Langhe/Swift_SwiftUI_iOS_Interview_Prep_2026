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

