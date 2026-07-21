# Day 34: Request And Response Modeling

## Why Modeling Matters

Good networking code separates:

- Request models
- Response DTOs
- Domain models
- Display models
- Error models

This prevents weak API shapes from leaking into the app.

## Request Model

```swift
struct UpdateProfileRequest: Encodable {
    let displayName: String
    let bio: String?
    let visibility: String
}
```

Use request models to:

- Avoid hand-built dictionaries.
- Keep encoding testable.
- Control API field names.
- Avoid accidentally sending UI-only state.

## Response DTO

```swift
struct ProfileResponse: Decodable {
    let id: String
    let display_name: String
    let bio: String?
    let visibility: String
}
```

DTOs represent transport shape, not necessarily app truth.

## Domain Model

```swift
struct Profile: Equatable, Identifiable {
    let id: ProfileID
    var displayName: String
    var bio: String
    var isPublic: Bool
}
```

Mapping:

```swift
extension Profile {
    init(response: ProfileResponse) throws {
        guard let id = ProfileID(rawValue: response.id) else {
            throw MappingError.invalidID
        }

        self.id = id
        self.displayName = response.display_name
        self.bio = response.bio ?? ""
        self.isPublic = response.visibility == "public"
    }
}
```

## Why Separate DTO And Domain

Separate when:

- API uses weak strings.
- API has nullable fields.
- App needs invariants.
- Response includes extra fields.
- UI requires stable domain semantics.
- Backend can change independently.

Keep same model only when:

- App is small.
- API shape exactly matches app needs.
- No invariants or mapping value exists.

## Endpoint Modeling

```swift
enum Endpoint {
    case products
    case product(Product.ID)
    case createOrder

    var path: String {
        switch self {
        case .products:
            "products"
        case .product(let id):
            "products/\(id.rawValue)"
        case .createOrder:
            "orders"
        }
    }

    var method: HTTPMethod {
        switch self {
        case .products, .product:
            .get
        case .createOrder:
            .post
        }
    }
}
```

## API Client Request Builder

```swift
struct RequestBuilder {
    let baseURL: URL
    let encoder: JSONEncoder

    func request<Body: Encodable>(
        endpoint: Endpoint,
        body: Body? = Optional<EmptyBody>.none
    ) throws -> URLRequest {
        var request = URLRequest(url: baseURL.appending(path: endpoint.path))
        request.httpMethod = endpoint.method.rawValue
        request.setValue("application/json", forHTTPHeaderField: "Accept")

        if let body {
            request.setValue("application/json", forHTTPHeaderField: "Content-Type")
            request.httpBody = try encoder.encode(body)
        }

        return request
    }
}
```

## Empty Responses

Handle `204 No Content`.

```swift
struct EmptyResponse: Decodable {}

func decode<Response: Decodable>(_ type: Response.Type, from data: Data) throws -> Response {
    if Response.self == EmptyResponse.self, data.isEmpty {
        return EmptyResponse() as! Response
    }

    return try decoder.decode(Response.self, from: data)
}
```

Avoid force casts in production generic helpers when possible; this is conceptual.

## Date And Key Strategies

```swift
let decoder = JSONDecoder()
decoder.keyDecodingStrategy = .convertFromSnakeCase
decoder.dateDecodingStrategy = .iso8601
```

Use deliberately. Global key conversion can be convenient but can surprise with acronyms or custom keys.

## Testing Models

```swift
@Test
func profileResponse_mapsToDomain() throws {
    let response = ProfileResponse(
        id: "123",
        display_name: "Ashish",
        bio: nil,
        visibility: "public"
    )

    let profile = try Profile(response: response)

    #expect(profile.bio == "")
    #expect(profile.isPublic)
}
```

## Senior iOS Engineer Perspective

Modeling questions:

- Is this transport or domain?
- Which fields are optional externally but required internally?
- Where are invalid IDs rejected?
- Does the API use snake case, dates, enums, polymorphism?
- How are empty responses handled?
- Are request models tested?

## Common Mistakes

- Using dictionaries for request bodies.
- Using DTOs directly in SwiftUI.
- No mapping tests.
- Force-unwrapping decoded fields.
- Ignoring empty responses.
- Global decoder strategies causing hidden bugs.
- Backend error models mixed with display models.

## Interview Notes

Junior:

Request models encode data sent to the API. Response models decode data returned by the API.

Mid-level:

Separate DTOs from domain models when API shape differs from app invariants.

Senior:

I model networking boundaries explicitly: request DTOs, response DTOs, domain mapping, error models, empty responses, decoder strategies, and tests for invalid payloads.

## Practice

1. Create request/response/domain models for profile update.
2. Add mapping validation for invalid IDs.
3. Handle a 204 response.
4. Test decoding snake-case JSON.
