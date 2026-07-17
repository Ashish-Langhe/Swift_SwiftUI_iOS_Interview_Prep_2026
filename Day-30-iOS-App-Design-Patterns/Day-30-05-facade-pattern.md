# Day 30: Facade Pattern

## What Facade Pattern Means

A Facade provides a simple interface over a complex subsystem.

It hides complexity from callers.

Example:

```swift
protocol ImageUploading {
    func uploadProfileImage(_ image: UIImage) async throws -> URL
}
```

Behind this simple method, the implementation may:

- Compress image
- Create upload token
- Upload to storage
- Register image URL with backend
- Track analytics
- Clean temporary files

## Why Facade Matters

Facade reduces coupling to complicated flows.

Without facade:

```swift
let compressed = try imageCompressor.compress(image)
let token = try await uploadTokenService.token()
let url = try await storageClient.upload(compressed, token: token)
try await profileAPI.updateImageURL(url)
analytics.track("profile_image_uploaded")
```

With facade:

```swift
let url = try await imageUploadFacade.uploadProfileImage(image)
```

## Facade Example

```swift
struct ProfileImageUploadFacade: ImageUploading {
    let compressor: ImageCompressor
    let tokenService: UploadTokenService
    let storageClient: StorageClient
    let profileAPI: ProfileAPI
    let analytics: AnalyticsClient

    func uploadProfileImage(_ image: UIImage) async throws -> URL {
        let data = try compressor.compress(image)
        let token = try await tokenService.uploadToken()
        let url = try await storageClient.upload(data, token: token)
        try await profileAPI.updateProfileImage(url)
        analytics.track("profile_image_uploaded")
        return url
    }
}
```

## ViewModel Uses Facade

```swift
@Observable
@MainActor
final class ProfileImageViewModel {
    var state: UploadState = .idle

    private let uploader: ImageUploading

    init(uploader: ImageUploading) {
        self.uploader = uploader
    }

    func upload(_ image: UIImage) async {
        state = .uploading

        do {
            let url = try await uploader.uploadProfileImage(image)
            state = .uploaded(url)
        } catch {
            state = .failed("Image upload failed.")
        }
    }
}
```

## Facade vs Service vs Repository

Service:

- One technical capability.
- Example: `StorageClient.upload`.

Repository:

- Domain data access.
- Example: `ProfileRepository.profile`.

Facade:

- Simplifies a complex subsystem/workflow.
- Example: `ProfileImageUploadFacade.uploadProfileImage`.

## When To Use Facade

Use when:

- Caller needs a simple API over many steps.
- Subsystem has multiple dependencies.
- You want to hide vendor SDK complexity.
- You want easier testing at the feature boundary.
- Workflow details should not leak.

Skip when:

- There is only one simple service call.
- Facade name hides too much unrelated behavior.
- It becomes a god object.

## Real iOS Use Cases

Good facade candidates:

- Image upload pipeline
- Analytics event grouping
- Push notification registration
- Authentication SDK wrapper
- Payment SDK wrapper
- Document export/share pipeline
- Feature flag evaluation

## Common Mistakes

- Facade becomes a giant manager.
- Hiding important failure modes.
- Combining unrelated workflows.
- Facade returns UI strings.
- No tests for workflow order.
- Leaking vendor SDK types through facade API.

## Senior Artifact

```text
Artifact: Facade Review
Subsystem:
Simple API:
Hidden steps:
Dependencies:
Failure modes:
Vendor leakage:
Tests:
Overreach risk:
```

## Interview Notes

Junior:

Facade gives a simple interface to complex code.

Mid-level:

Use facade to hide multi-step workflows or SDK complexity from ViewModels.

Senior:

I use facades to simplify subsystem boundaries while preserving meaningful errors and tests. I avoid facades that become vague managers or hide unrelated responsibilities.

## Practice

1. Build a profile image upload facade.
2. Wrap a payment SDK behind a facade.
3. Identify failure modes hidden by a facade.
4. Explain facade vs repository.
