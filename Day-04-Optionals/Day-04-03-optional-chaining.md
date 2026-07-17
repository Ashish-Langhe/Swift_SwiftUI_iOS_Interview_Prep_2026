# Day 4: Optional Chaining

## Core Idea

Optional chaining lets you access properties, methods, and subscripts on an optional. If any part is nil, the whole chain returns nil.

```swift
let city = user.address?.city
```

If `address` is nil, `city` becomes nil.

## Basic Example

```swift
struct Address {
    let city: String
}

struct User {
    let address: Address?
}

let user = User(address: Address(city: "Pune"))
let city = user.address?.city
```

`city` is `String?`, not `String`.

## Method Calls

```swift
let text: String? = "Swift"
let count = text?.count
```

`count` is `Int?`.

## Chaining Multiple Levels

```swift
struct Company {
    let address: Address?
}

struct Employee {
    let company: Company?
}

let employee = Employee(company: nil)
let city = employee.company?.address?.city
```

If `company` or `address` is nil, `city` is nil.

## Optional Chaining With Nil-Coalescing

```swift
let displayCity = employee.company?.address?.city ?? "Unknown city"
```

This is common for display fallback.

## Optional Chaining With Functions

```swift
final class AnalyticsService {
    func track(_ event: String) {
        print(event)
    }
}

let analytics: AnalyticsService? = AnalyticsService()
analytics?.track("screen_view")
```

If `analytics` is nil, the method is not called.

## Assignment Through Optional Chaining

```swift
final class Profile {
    var name: String = "Guest"
}

var profile: Profile? = Profile()
profile?.name = "Aarav"
```

The assignment happens only if `profile` is not nil.

## Real iOS Use Cases

### API Response

```swift
struct ProfileResponse: Decodable {
    let user: UserResponse?
}

struct UserResponse: Decodable {
    let address: AddressResponse?
}

struct AddressResponse: Decodable {
    let city: String?
}

let city = response.user?.address?.city ?? "Unknown"
```

### UIKit Delegate

```swift
protocol LoginDelegate: AnyObject {
    func didLogin()
}

final class LoginViewController {
    weak var delegate: LoginDelegate?

    func loginCompleted() {
        delegate?.didLogin()
    }
}
```

### SwiftUI Optional State

```swift
let title = selectedUser?.name ?? "No user selected"
```

## Junior-Level Interview Answer

Optional chaining lets us access values inside optionals safely using `?.`.

## Mid-Level Interview Answer

Optional chaining returns nil if any optional in the chain is nil. It is useful for nested models and optional delegates.

## Senior-Level Interview Answer

Optional chaining is concise and safe, but long chains can hide weak domain modeling. If a chain like `order.customer?.profile?.address?.city` appears in many places, I may introduce a view model, computed property, or domain method that gives the UI a clearer value.

## Quick Interview Notes

- `?.` performs optional chaining.
- If any part is nil, the result is nil.
- Optional chaining result is usually optional.
- It works with properties, methods, and subscripts.
- Combine with `??` for display fallback.

## Practice Questions

1. What does `?.` do?
2. What is the result type of optional chaining?
3. Can optional chaining call methods?
4. What happens if the optional is nil?
5. When can long optional chains become a design smell?

