# Day 21: Codable

## What Codable Is

`Codable` is a type alias for `Encodable & Decodable`.

```swift
struct User: Codable {
    let id: String
    let name: String
}
```

It lets Swift encode and decode values to formats such as JSON.

## Basic Decoding

```swift
let user = try JSONDecoder().decode(User.self, from: data)
```

## Basic Encoding

```swift
let data = try JSONEncoder().encode(user)
```

## Custom Coding Keys

APIs often use snake_case while Swift uses camelCase.

```swift
struct UserResponse: Decodable {
    let userID: String
    let fullName: String

    enum CodingKeys: String, CodingKey {
        case userID = "user_id"
        case fullName = "full_name"
    }
}
```

## Decoder Strategies

```swift
let decoder = JSONDecoder()
decoder.keyDecodingStrategy = .convertFromSnakeCase
decoder.dateDecodingStrategy = .iso8601
```

Strategies are useful, but be careful with APIs that are inconsistent.

## DTO vs Domain Model

Senior engineers often separate transport shape from domain shape.

```swift
struct UserResponse: Decodable {
    let id: String
    let fullName: String?
}

struct User {
    let id: String
    let name: String
}

extension User {
    init(response: UserResponse) throws {
        guard let fullName = response.fullName, !fullName.isEmpty else {
            throw MappingError.missingName
        }
        self.id = response.id
        self.name = fullName
    }
}
```

## Real iOS Use Cases

- Decode network responses.
- Encode request bodies.
- Store cached models.
- Persist settings.
- Parse local JSON fixtures for tests.

## Handling Optional Fields

```swift
struct ProductResponse: Decodable {
    let id: String
    let title: String
    let subtitle: String?
}
```

Use optionals only when absence is valid. Do not make everything optional just to avoid decoding failures.

## Senior iOS Engineer Artifact

```text
Artifact: Codable Boundary Map
Topic: Codable
Transport model: API response/request type
Domain model: app-safe type
Optional policy: valid absence vs invalid payload
Date/key strategy: explicit or decoder-level
Error handling: decoding vs mapping vs server error
Test fixtures: valid, missing field, invalid type, extra field
```

Senior lens:

- Keep backend DTOs away from UI when the domain needs stronger guarantees.
- Treat decoding errors as data-quality signals.
- Test optional and invalid payloads.
- Be intentional with date and key strategies.

## More Coding Examples

### Example 1: Encode Request Body

```swift
struct LoginRequest: Encodable {
    let email: String
    let password: String
}

request.httpBody = try JSONEncoder().encode(LoginRequest(email: email, password: password))
```

### Example 2: Decode Fixture In Tests

```swift
let data = try Data(contentsOf: fixtureURL)
let response = try JSONDecoder().decode(UserResponse.self, from: data)
```

## Common Mistakes

- Making all fields optional.
- Exposing DTOs directly to UI.
- Ignoring date decoding strategy.
- Confusing decoding failure with network failure.
- Force-trying fixture decoding in production code.
- Using `convertFromSnakeCase` without checking edge cases.

## Interview Guide

Junior:

Codable lets Swift types encode and decode data such as JSON.

Mid-level:

Use `CodingKeys`, decoder strategies, and optionals carefully to match API payloads.

Senior:

Codable sits at a boundary. I separate transport models from domain models when needed, validate payload assumptions, and test both valid and invalid JSON.

## Practice

1. Decode a JSON user response.
2. Add custom coding keys.
3. Map a DTO into a domain model.
4. Write test fixtures for missing and invalid fields.
