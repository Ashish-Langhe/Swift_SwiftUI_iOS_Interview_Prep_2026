# Day 37: Deeper Swift Problem Solving

These questions test deeper Swift understanding. They are useful for mid-level, senior, and architect interviews because they reveal how well a candidate understands value semantics, copy-on-write, protocols, generics, closures, error modeling, concurrency, and API design.

## Value Semantics And Copy Behavior

### Problem 1: Explain And Predict Struct Mutation

```swift
struct Counter {
    var value: Int
}

var first = Counter(value: 1)
var second = first
second.value += 1

print(first.value)
print(second.value)
```

Expected:

```text
1
2
```

Why: structs are value types. Assigning `first` to `second` creates an independent value.

### Problem 2: Toggle Item By ID Without Mutating Original Array

```swift
struct Todo: Identifiable, Equatable {
    let id: String
    let title: String
    var isDone: Bool
}

func toggledTodo(id: String, in todos: [Todo]) -> [Todo] {
    todos.map { todo in
        guard todo.id == id else { return todo }
        var updated = todo
        updated.isDone.toggle()
        return updated
    }
}
```

Senior discussion:

- This is clean for UI state.
- For huge arrays, consider index lookup or identified collections.
- For diffable UI, stable IDs matter.

### Problem 3: Mutating Method vs Pure Function

```swift
struct Cart {
    private(set) var items: [String] = []

    mutating func add(_ item: String) {
        items.append(item)
    }

    func adding(_ item: String) -> Cart {
        var copy = self
        copy.add(item)
        return copy
    }
}
```

What it tests:

- API design.
- Value mutation.
- Functional update style.

## Reference Semantics And Identity

### Problem 4: Predict Class Mutation

```swift
final class Box {
    var value: Int

    init(value: Int) {
        self.value = value
    }
}

let first = Box(value: 1)
let second = first
second.value = 2

print(first.value)
```

Expected:

```text
2
```

Why: classes are reference types. Both constants point to the same instance.

### Problem 5: Identity vs Equality

```swift
final class UserSession: Equatable {
    let id: String

    init(id: String) {
        self.id = id
    }

    static func == (lhs: UserSession, rhs: UserSession) -> Bool {
        lhs.id == rhs.id
    }
}

let a = UserSession(id: "1")
let b = UserSession(id: "1")

print(a == b)
print(a === b)
```

Expected:

```text
true
false
```

Senior discussion:

- `==` means logical equality.
- `===` means same object identity.
- UIKit often cares about identity; domain models often care about equality.

## Protocols And Generics

### Problem 6: Generic Duplicate Finder

```swift
func firstDuplicate<T: Hashable>(in values: [T]) -> T? {
    var seen = Set<T>()

    for value in values {
        if !seen.insert(value).inserted {
            return value
        }
    }

    return nil
}
```

What it tests:

- Generics.
- `Hashable`.
- Set insertion result.

### Problem 7: Generic Grouping By Key

```swift
func group<Value, Key: Hashable>(
    _ values: [Value],
    by key: (Value) -> Key
) -> [Key: [Value]] {
    var result: [Key: [Value]] = [:]

    for value in values {
        result[key(value), default: []].append(value)
    }

    return result
}
```

Usage:

```swift
struct Expense {
    let category: String
    let amount: Decimal
}

let grouped = group(expenses, by: \.category)
```

Senior discussion:

- This is reusable because the grouping key is injected.
- The key must be `Hashable` because it is used in a dictionary.

### Problem 8: Protocol-Based Formatter

```swift
protocol PriceFormatting {
    func string(from amount: Decimal, currencyCode: String) -> String
}

struct SimplePriceFormatter: PriceFormatting {
    func string(from amount: Decimal, currencyCode: String) -> String {
        "\(currencyCode) \(amount)"
    }
}

struct ProductRowMapper {
    let formatter: PriceFormatting

    func map(product: Product) -> ProductRow {
        ProductRow(
            id: product.id,
            title: product.name,
            priceText: formatter.string(
                from: product.price,
                currencyCode: product.currencyCode
            )
        )
    }
}
```

Senior discussion:

- The mapper is testable.
- The formatter can be mocked.
- The UI does not own formatting policy.

## Closures And Captures

### Problem 9: Fix Retain Cycle

Bad:

```swift
final class ProfileViewModel {
    var onUpdate: (() -> Void)?
}

final class ProfileViewController {
    let viewModel = ProfileViewModel()

    func bind() {
        viewModel.onUpdate = {
            self.render()
        }
    }

    func render() {}
}
```

Better:

```swift
func bind() {
    viewModel.onUpdate = { [weak self] in
        self?.render()
    }
}
```

Senior discussion:

- View controller owns view model.
- View model owns closure.
- Closure strongly owning view controller creates a cycle.

### Problem 10: Capture Value At Time Of Closure Creation

```swift
var count = 0
let printCurrent = { [count] in
    print(count)
}

count = 10
printCurrent()
```

Expected:

```text
0
```

Why: `[count]` captures the value at closure creation time.

## Error Modeling

### Problem 11: Typed Validation Result

```swift
enum PasswordValidationError: Error, Equatable {
    case tooShort(minimum: Int)
    case missingDigit
    case missingUppercaseLetter
}

func validatePassword(_ password: String) throws {
    guard password.count >= 8 else {
        throw PasswordValidationError.tooShort(minimum: 8)
    }

    guard password.contains(where: { $0.isNumber }) else {
        throw PasswordValidationError.missingDigit
    }

    guard password.contains(where: { $0.isUppercase }) else {
        throw PasswordValidationError.missingUppercaseLetter
    }
}
```

Senior discussion:

- Typed errors make tests clear.
- UI can map each error to a specific message.
- Do not return plain strings as business errors.

### Problem 12: Result-Based API

```swift
func validateEmail(_ email: String) -> Result<String, EmailError> {
    guard !email.isEmpty else { return .failure(.empty) }
    guard email.contains("@") else { return .failure(.invalidFormat) }
    return .success(email)
}

enum EmailError: Error {
    case empty
    case invalidFormat
}
```

When to use:

- `throws` is good for call chains.
- `Result` is good when storing or passing success/failure as a value.

## Collection Performance

### Problem 13: Array Lookup vs Set Lookup

```swift
func containsBlockedUserSlow(userID: String, blockedIDs: [String]) -> Bool {
    blockedIDs.contains(userID)
}

func containsBlockedUserFast(userID: String, blockedIDs: Set<String>) -> Bool {
    blockedIDs.contains(userID)
}
```

Complexity:

- Array contains: O(n)
- Set contains: average O(1)

Senior discussion:

If this check happens once, array may be fine. If it happens for every visible cell, convert to `Set`.

### Problem 14: Avoid Repeated Formatter Creation

Bad:

```swift
func priceText(_ amount: Decimal) -> String {
    let formatter = NumberFormatter()
    formatter.numberStyle = .currency
    return formatter.string(from: amount as NSDecimalNumber) ?? ""
}
```

Better:

```swift
final class CurrencyFormatter {
    private let formatter: NumberFormatter = {
        let formatter = NumberFormatter()
        formatter.numberStyle = .currency
        return formatter
    }()

    func string(from amount: Decimal) -> String {
        formatter.string(from: amount as NSDecimalNumber) ?? ""
    }
}
```

Senior discussion:

Formatters can be expensive. Avoid creating them inside cell configuration.

## Concurrency Logic

### Problem 15: Prevent Stale Search Results

```swift
@MainActor
final class SearchViewModel {
    private var currentQuery = ""
    private let service: SearchService

    init(service: SearchService) {
        self.service = service
    }

    func search(_ query: String) async {
        currentQuery = query

        do {
            let results = try await service.search(query)

            guard currentQuery == query else {
                return
            }

            render(results)
        } catch {
            guard currentQuery == query else {
                return
            }

            renderError(error)
        }
    }

    private func render(_ results: [SearchResult]) {}
    private func renderError(_ error: Error) {}
}
```

Senior discussion:

This protects against stale responses. A better production version may also cancel the previous task.

### Problem 16: Actor Counter

```swift
actor DownloadCounter {
    private var completedCount = 0

    func increment() {
        completedCount += 1
    }

    func value() -> Int {
        completedCount
    }
}
```

What it tests:

- Shared mutable state.
- Actor isolation.
- Data-race safety.

## API Design Questions

### Problem 17: Design A Safe Subscript

```swift
extension Collection {
    subscript(safe index: Index) -> Element? {
        indices.contains(index) ? self[index] : nil
    }
}
```

Senior discussion:

This works for any collection, but `indices.contains(index)` may not be O(1) for every collection. For `Array`, it is fine.

### Problem 18: Design A Type-Safe Identifier

```swift
struct UserID: Hashable, RawRepresentable {
    let rawValue: String
}

struct OrderID: Hashable, RawRepresentable {
    let rawValue: String
}

func loadUser(id: UserID) {}
```

Why it matters:

- Prevents passing `OrderID` where `UserID` is expected.
- Helps large codebases avoid stringly typed bugs.

## Senior Swift Interview Checklist

- Can explain value vs reference semantics.
- Can design stable identity.
- Can choose protocol/generic abstraction wisely.
- Can identify closure retain cycles.
- Can model errors with enums.
- Can discuss collection complexity.
- Can write actor-isolated shared state.
- Can avoid stale async results.
- Can make code testable with injected dependencies.
- Can avoid overengineering simple problems.

## Architect-Level Takeaway

Deep Swift problem solving is not about using the fanciest feature. It is about choosing the smallest correct tool that makes invalid states hard, behavior testable, ownership clear, and future changes manageable.

