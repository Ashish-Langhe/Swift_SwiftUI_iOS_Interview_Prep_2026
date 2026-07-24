# Day 37: Swift Basics Coding Drills

These questions test whether a candidate can write clean Swift without depending on memorized algorithm templates. They are especially useful for junior iOS interviews, but senior candidates should still answer them with precision, edge cases, and idiomatic Swift.

## Variables, Constants, And Type Inference

1. Declare a constant `appName` and a variable `launchCount`.
   - Tests: `let` vs `var`.
   - Good answer: use `let` by default, `var` only when mutation is needed.

```swift
let appName = "Interview Prep"
var launchCount = 0
launchCount += 1
```

2. Explain why this code fails:

```swift
let score = 10
score = 20
```

Answer: `score` is declared with `let`, so it cannot be reassigned.

3. Write a function that returns the full name from first and last name.

```swift
func fullName(firstName: String, lastName: String) -> String {
    "\(firstName) \(lastName)"
}
```

4. Convert a string `"42"` into an integer safely.

```swift
func parseInt(_ text: String) -> Int? {
    Int(text)
}
```

Senior note: never force unwrap user input.

5. Convert an optional integer into display text.

```swift
func displayCount(_ count: Int?) -> String {
    guard let count else { return "No count" }
    return "\(count)"
}
```

## Optionals

6. Safely unwrap an optional username.

```swift
func greeting(for username: String?) -> String {
    guard let username, !username.isEmpty else {
        return "Hello, Guest"
    }
    return "Hello, \(username)"
}
```

7. Chain optional properties safely.

```swift
struct Address {
    let city: String?
}

struct User {
    let address: Address?
}

func cityName(for user: User?) -> String {
    user?.address?.city ?? "Unknown City"
}
```

8. Replace force unwrap with safe handling.

Bad:

```swift
let value = Int(text)!
```

Better:

```swift
guard let value = Int(text) else {
    return
}
```

9. Write a function that returns the first non-empty optional string.

```swift
func firstNonEmpty(_ values: [String?]) -> String? {
    for value in values {
        if let value, !value.isEmpty {
            return value
        }
    }
    return nil
}
```

10. Compact optional values from an array.

```swift
func removeNilValues(_ values: [Int?]) -> [Int] {
    values.compactMap { $0 }
}
```

## Strings

11. Count characters in a Swift string.

```swift
func characterCount(_ text: String) -> Int {
    text.count
}
```

Swift note: `String.count` counts user-perceived characters, not bytes.

12. Check if a string is empty after trimming spaces.

```swift
func isBlank(_ text: String) -> Bool {
    text.trimmingCharacters(in: .whitespacesAndNewlines).isEmpty
}
```

13. Capitalize each name in an array.

```swift
func capitalizedNames(_ names: [String]) -> [String] {
    names.map { $0.capitalized }
}
```

14. Filter names that start with a prefix.

```swift
func namesStarting(with prefix: String, in names: [String]) -> [String] {
    names.filter { $0.hasPrefix(prefix) }
}
```

15. Count words in a sentence.

```swift
func wordCount(_ sentence: String) -> Int {
    sentence
        .split(whereSeparator: { $0.isWhitespace })
        .count
}
```

## Arrays

16. Return only even numbers.

```swift
func evenNumbers(from numbers: [Int]) -> [Int] {
    numbers.filter { $0 % 2 == 0 }
}
```

17. Square every number.

```swift
func squared(_ numbers: [Int]) -> [Int] {
    numbers.map { $0 * $0 }
}
```

18. Sum all numbers.

```swift
func sum(_ numbers: [Int]) -> Int {
    numbers.reduce(0, +)
}
```

19. Return the largest number safely.

```swift
func largest(_ numbers: [Int]) -> Int? {
    numbers.max()
}
```

20. Split numbers into positive and negative.

```swift
func splitBySign(_ numbers: [Int]) -> (positive: [Int], negative: [Int]) {
    var positive: [Int] = []
    var negative: [Int] = []

    for number in numbers {
        if number >= 0 {
            positive.append(number)
        } else {
            negative.append(number)
        }
    }

    return (positive, negative)
}
```

## Dictionaries

21. Count character frequencies.

```swift
func characterFrequencies(_ text: String) -> [Character: Int] {
    var counts: [Character: Int] = [:]
    for character in text {
        counts[character, default: 0] += 1
    }
    return counts
}
```

22. Group names by first letter.

```swift
func groupNamesByFirstLetter(_ names: [String]) -> [Character: [String]] {
    var groups: [Character: [String]] = [:]

    for name in names {
        guard let first = name.first else { continue }
        groups[first, default: []].append(name)
    }

    return groups
}
```

23. Merge two dictionaries where second wins.

```swift
func mergeSettings(
    defaults: [String: String],
    overrides: [String: String]
) -> [String: String] {
    defaults.merging(overrides) { _, new in new }
}
```

24. Find key for maximum value.

```swift
func topScorer(_ scores: [String: Int]) -> String? {
    scores.max { $0.value < $1.value }?.key
}
```

25. Count words in a sentence.

```swift
func wordFrequencies(_ sentence: String) -> [String: Int] {
    var counts: [String: Int] = [:]
    for word in sentence.lowercased().split(whereSeparator: { $0.isWhitespace }) {
        counts[String(word), default: 0] += 1
    }
    return counts
}
```

## Sets

26. Check if two arrays share any value.

```swift
func hasOverlap(_ lhs: [Int], _ rhs: [Int]) -> Bool {
    !Set(lhs).isDisjoint(with: Set(rhs))
}
```

27. Find unique values.

```swift
func uniqueValues(_ numbers: [Int]) -> [Int] {
    Array(Set(numbers))
}
```

28. Find common values.

```swift
func commonValues(_ lhs: [Int], _ rhs: [Int]) -> [Int] {
    Array(Set(lhs).intersection(Set(rhs)))
}
```

29. Find values in first array but not second.

```swift
func difference(_ lhs: [Int], _ rhs: [Int]) -> [Int] {
    Array(Set(lhs).subtracting(Set(rhs)))
}
```

30. Check if all required permissions exist.

```swift
func hasRequiredPermissions(
    granted: Set<String>,
    required: Set<String>
) -> Bool {
    required.isSubset(of: granted)
}
```

## Tuples And Return Values

31. Return min and max from an array.

```swift
func minMax(_ numbers: [Int]) -> (min: Int, max: Int)? {
    guard let min = numbers.min(), let max = numbers.max() else {
        return nil
    }
    return (min, max)
}
```

32. Return success flag and message.

```swift
func validateAge(_ age: Int) -> (isValid: Bool, message: String) {
    age >= 18
        ? (true, "Allowed")
        : (false, "Must be at least 18")
}
```

33. Swap tuple values.

```swift
func swapped<T, U>(_ pair: (T, U)) -> (U, T) {
    (pair.1, pair.0)
}
```

## Enums

34. Model API loading state.

```swift
enum LoadingState<Value> {
    case idle
    case loading
    case loaded(Value)
    case failed(String)
}
```

35. Convert status into display text.

```swift
enum PaymentStatus {
    case pending
    case completed
    case failed

    var title: String {
        switch self {
        case .pending: return "Pending"
        case .completed: return "Completed"
        case .failed: return "Failed"
        }
    }
}
```

36. Use enum associated values for validation errors.

```swift
enum FormError: Error {
    case missingField(String)
    case invalidEmail
    case passwordTooShort(minimum: Int)
}
```

## Structs

37. Create a value type for product.

```swift
struct Product: Identifiable, Hashable {
    let id: String
    var name: String
    var price: Decimal
}
```

38. Toggle favorite using value semantics.

```swift
struct ProductRow: Identifiable, Hashable {
    let id: String
    let title: String
    var isFavorite: Bool

    mutating func toggleFavorite() {
        isFavorite.toggle()
    }
}
```

39. Update an item in an array by ID.

```swift
func toggledFavorite(id: String, rows: [ProductRow]) -> [ProductRow] {
    rows.map { row in
        guard row.id == id else { return row }
        var updated = row
        updated.toggleFavorite()
        return updated
    }
}
```

## Basic Error Handling

40. Throw validation errors.

```swift
enum LoginValidationError: Error {
    case emptyEmail
    case invalidEmail
    case emptyPassword
}

func validateLogin(email: String, password: String) throws {
    guard !email.isEmpty else { throw LoginValidationError.emptyEmail }
    guard email.contains("@") else { throw LoginValidationError.invalidEmail }
    guard !password.isEmpty else { throw LoginValidationError.emptyPassword }
}
```

41. Convert throwing result to user message.

```swift
func loginMessage(email: String, password: String) -> String {
    do {
        try validateLogin(email: email, password: password)
        return "Valid"
    } catch LoginValidationError.emptyEmail {
        return "Email is required"
    } catch LoginValidationError.invalidEmail {
        return "Email is invalid"
    } catch LoginValidationError.emptyPassword {
        return "Password is required"
    } catch {
        return "Unknown error"
    }
}
```

## Junior Interview Checklist

- Can explain `let` vs `var`.
- Can safely unwrap optionals.
- Can use `map`, `filter`, `reduce`, and loops.
- Can choose array vs dictionary vs set.
- Can model simple state with enum.
- Can write functions with clear inputs and outputs.
- Can avoid force unwraps.
- Can explain simple edge cases.

## Common Beginner Mistakes

- Force unwrapping parsed input.
- Using array search when dictionary lookup is better.
- Forgetting empty arrays.
- Mutating when a pure function would be clearer.
- Returning unclear tuple values without labels.
- Using `Any` instead of proper types.
- Not using enums for limited states.

