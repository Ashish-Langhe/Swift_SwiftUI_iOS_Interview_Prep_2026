# Day 34: Networking Interview Guide

## One-Minute Senior Answer

iOS networking should be designed as a layered, testable boundary. I use URLSession for data, upload, download, background, and streaming tasks; model REST requests and responses explicitly; validate HTTP status before decoding; store tokens securely in Keychain; centralize authentication headers and token refresh; apply retry/backoff only to safe transient failures; use multipart and download tasks for files; treat reachability as advisory; and keep API transport separate from repositories/domain models.

## Core Mental Models

```text
URLSession:
Transfer engine

URLRequest:
Method, headers, body, cache, timeout

REST:
Resource semantics over HTTP

DTO:
Transport shape

Domain model:
App meaning

Authenticator:
Token/session handling

Repository:
Domain data boundary

Retry policy:
Transient failure handling with safety

Reachability:
Advisory network path signal
```

## Junior Questions

What is URLSession?

Foundation API for network requests, uploads, downloads, and related tasks.

What is a REST API?

An API using HTTP methods and resource URLs to read or modify server data.

What is an HTTP status code?

A numeric response status such as 200 success, 401 unauthorized, or 500 server error.

## Mid-Level Questions

How do you make a POST request?

Create `URLRequest`, set method to `POST`, set headers like `Content-Type`, encode request body, call `URLSession.data(for:)`, validate response, decode data.

How do you handle authentication?

Store tokens securely, add `Authorization: Bearer` header centrally, handle 401 through token refresh, and clear session if refresh fails.

How do you upload a file?

Use multipart/form-data for fields plus files, and prefer `upload(for:fromFile:)` for large uploads.

## Senior Questions

How do you design an API client?

I separate request building, auth, transport, response validation, decoding, error mapping, retry, logging, and domain mapping. ViewModels depend on repositories/use cases, not raw URLSession.

How do you implement token refresh safely?

Use a centralized authenticator, often an actor, to serialize refresh calls and prevent refresh storms. Retry the original request once, store tokens in Keychain, and handle refresh failure globally.

How do you decide retry behavior?

Retry only transient failures, use bounded exponential backoff with jitter, respect cancellation and Retry-After, and never blindly retry non-idempotent writes like payments.

How do you use reachability?

As advisory UI/policy signal, not as proof that a request will succeed. Requests still need normal error handling.

## Senior Artifact: Networking Architecture Review

```text
Feature:
Endpoints:
Request models:
Response DTOs:
Domain mapping:
Auth required:
Token refresh:
Retry policy:
Idempotency:
Uploads/downloads:
Reachability behavior:
Error mapping:
Logging/redaction:
Tests:
Risks:
```

## Real Scenario: Checkout

Networking concerns:

- `POST /orders`
- Auth header
- Idempotency key
- Request DTO for cart/payment
- Response DTO mapped to `Order`
- 401 token refresh
- 422 validation errors
- No blind retry for payment/order write
- Request ID logging
- Display-safe error mapping

Strong design:

```text
CheckoutViewModel -> CheckoutUseCase -> OrderRepository -> APIClient -> URLSession
```

## Common Interview Traps

- Decoding before checking status.
- Storing tokens in UserDefaults.
- Retrying all failures.
- No answer for token refresh race conditions.
- Treating reachability as guaranteed connectivity.
- Uploading huge files from memory.
- ViewModels building requests directly.
- Showing raw backend errors to users.

## Strong Closing Answer

Networking is not just calling `URLSession`. It is transport design, security, state, retries, error contracts, testability, and user experience. Senior iOS networking code is centralized where it should be, but still domain-aware at repository/use-case boundaries.

## Practice Prompts

1. Design an API client for a shopping app.
2. Explain token refresh with concurrent requests.
3. Design retry policy for search vs checkout.
4. Build multipart upload for profile image.
5. Explain reachability and offline mode.
6. Model DTO/domain mapping for product response.
