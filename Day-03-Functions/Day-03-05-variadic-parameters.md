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

