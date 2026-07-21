# Day 34: REST APIs

## What REST APIs Are

REST APIs expose resources through HTTP endpoints.

Common resources:

- `/users`
- `/products`
- `/orders`
- `/payments`
- `/sessions`

Common methods:

```text
GET:
Read resource

POST:
Create resource or perform command

PUT:
Replace resource

PATCH:
Partially update resource

DELETE:
Delete resource
```

## REST Example

```text
GET /products
GET /products/{id}
POST /orders
PATCH /profile
DELETE /cart/items/{id}
```

## HTTP Status Codes

Important groups:

```text
2xx:
Success

3xx:
Redirect

4xx:
Client/request/user/auth problem

5xx:
Server problem
```

Common codes:

- `200 OK`
- `201 Created`
- `204 No Content`
- `400 Bad Request`
- `401 Unauthorized`
- `403 Forbidden`
- `404 Not Found`
- `409 Conflict`
- `422 Validation Error`
- `429 Too Many Requests`
- `500 Server Error`
- `503 Service Unavailable`

## Request Example

```swift
struct CreateOrderRequest: Encodable {
    let cartID: String
    let paymentMethodID: String
}

func createOrder(_ requestBody: CreateOrderRequest) async throws -> OrderResponse {
    var request = URLRequest(url: baseURL.appending(path: "orders"))
    request.httpMethod = "POST"
    request.setValue("application/json", forHTTPHeaderField: "Content-Type")
    request.setValue("application/json", forHTTPHeaderField: "Accept")
    request.httpBody = try JSONEncoder().encode(requestBody)

    let (data, response) = try await session.data(for: request)
    return try decoder.decode(OrderResponse.self, from: try validate(data, response))
}
```

## Response Validation

```swift
func validate(_ data: Data, _ response: URLResponse) throws -> Data {
    guard let http = response as? HTTPURLResponse else {
        throw APIError.invalidResponse
    }

    switch http.statusCode {
    case 200..<300:
        return data
    case 401:
        throw APIError.unauthorized
    case 404:
        throw APIError.notFound
    case 409:
        throw APIError.conflict
    case 422:
        throw APIError.validationFailed(data)
    case 429:
        throw APIError.rateLimited
    case 500..<600:
        throw APIError.serverError
    default:
        throw APIError.unexpectedStatus(http.statusCode)
    }
}
```

Never treat every response body as success.

## Error Body Modeling

```swift
struct APIErrorResponse: Decodable {
    let code: String
    let message: String
    let fieldErrors: [String: String]?
}
```

Map API errors to domain/display errors:

```swift
func mapError(_ error: APIError) -> DisplayError {
    switch error {
    case .unauthorized:
        DisplayError(title: "Session Expired", message: "Please sign in again.")
    case .rateLimited:
        DisplayError(title: "Slow Down", message: "Try again in a moment.")
    default:
        DisplayError(title: "Something Went Wrong", message: "Please try again.")
    }
}
```

## Idempotency

Important for write APIs.

If a payment request times out, did the payment happen?

Use idempotency keys for retry-safe writes where backend supports it.

```swift
request.setValue(idempotencyKey.uuidString, forHTTPHeaderField: "Idempotency-Key")
```

Senior engineers always ask this for checkout, transfer, booking, and order creation.

## Pagination

Offset:

```text
GET /products?page=2&limit=50
```

Cursor:

```text
GET /products?cursor=abc123
```

Cursor pagination is usually better for changing datasets.

```swift
struct Page<Response: Decodable>: Decodable {
    let items: [Response]
    let nextCursor: String?
}
```

## Versioning

Common API version patterns:

```text
/v1/products
Accept: application/vnd.company.v1+json
```

iOS apps remain in the wild for years. API compatibility matters.

## Senior iOS Engineer Perspective

REST design questions:

- What is the resource?
- Is method semantics correct?
- What status codes can happen?
- Is the write idempotent?
- Is pagination stable?
- How are validation errors modeled?
- Does the app handle backward compatibility?
- Are transport DTOs separated from domain models?

## Common Mistakes

- Assuming only 200 success.
- Ignoring 204 no-content.
- Parsing error body as success DTO.
- Retrying non-idempotent writes.
- No pagination strategy.
- API DTOs used directly as domain models.
- User sees raw backend error text.

## Interview Notes

Junior:

REST APIs use HTTP methods and status codes to work with resources.

Mid-level:

Validate status codes, model request/response DTOs, and map API errors.

Senior:

I design REST clients around resource semantics, idempotency, pagination, error contracts, DTO/domain boundaries, compatibility, and retry safety.

## Practice

1. Model a `POST /orders` request.
2. Handle `401`, `422`, and `500` separately.
3. Add cursor pagination.
4. Explain idempotency for payment APIs.
