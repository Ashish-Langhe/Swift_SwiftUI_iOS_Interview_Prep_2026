# Day 3: Inout Parameters

## Core Idea

Function parameters are constants by default. You cannot modify them directly.

```swift
func increment(_ value: Int) {
    // value += 1
    // Error
}
```

Use `inout` when a function needs to modify the caller's variable.

```swift
func increment(_ value: inout Int) {
    value += 1
}

var count = 10
increment(&count)
print(count)
```

The `&` means the argument is passed for modification.

## How Inout Works Conceptually

Think of `inout` as:

1. Copy the value into the function.
2. Modify it inside the function.
3. Write the modified value back to the caller.

Swift enforces rules to keep this safe.

## Real iOS Use Cases

### Swapping Values

```swift
func swapValues<T>(_ first: inout T, _ second: inout T) {
    let temporary = first
    first = second
    second = temporary
}

var a = 10
var b = 20
swapValues(&a, &b)
```

Swift already has `swap(&a, &b)`, but this demonstrates the idea.

### Updating Draft State

```swift
struct ProfileDraft {
    var name: String
    var city: String
}

func normalize(_ draft: inout ProfileDraft) {
    draft.name = draft.name.trimmingCharacters(in: .whitespaces)
    draft.city = draft.city.trimmingCharacters(in: .whitespaces)
}

var draft = ProfileDraft(name: " Riya ", city: " Pune ")
normalize(&draft)
```

## Inout Vs Return New Value

Instead of:

```swift
func trim(_ text: inout String) {
    text = text.trimmingCharacters(in: .whitespaces)
}
```

Often prefer:

```swift
func trimmed(_ text: String) -> String {
    text.trimmingCharacters(in: .whitespaces)
}

let cleanText = trimmed(" Swift ")
```

Returning a new value is easier to reason about and test.

## Inout Restrictions

You can only pass variables, not constants or literals.

```swift
var value = 5
increment(&value)

let fixedValue = 5
// increment(&fixedValue)
// Error

// increment(&10)
// Error
```

## Inout And Side Effects

`inout` creates side effects because it modifies external state.

That is not always bad, but it should be intentional and obvious.

```swift
func applyDiscount(to price: inout Decimal) {
    price *= 0.9
}
```

The caller sees `&price`, which clearly signals mutation.

## Junior-Level Interview Answer

`inout` lets a function change the original variable passed by the caller. The caller uses `&` when passing the variable.

## Mid-Level Interview Answer

Function parameters are constants by default. `inout` allows controlled mutation of the caller's variable. It is useful for operations like swapping values or updating a mutable draft, but should not be overused.

## Senior-Level Interview Answer

`inout` is explicit shared mutation at the function boundary. I use it when mutation is the clearest API, such as low-level algorithms, performance-sensitive transformations, or update-style APIs. For most business logic, I prefer returning a new value because it is easier to test, compose, reason about, and use safely in concurrent contexts.

## Quick Interview Notes

- Parameters are constants by default.
- `inout` allows modifying caller variables.
- Callers pass arguments with `&`.
- You cannot pass constants or literals to `inout`.
- Prefer return values unless mutation is the clearest design.

## Practice Questions

1. Why do we need `inout`?
2. What does `&` mean at the call site?
3. Can you pass a `let` constant to an `inout` parameter?
4. When is returning a new value better than using `inout`?
5. Why should `inout` be used carefully?

