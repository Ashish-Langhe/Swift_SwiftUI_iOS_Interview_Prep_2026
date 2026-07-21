# Day 34: Multipart Upload

## What Multipart Upload Is

Multipart upload sends files and form fields in one HTTP request using `multipart/form-data`.

Use it for:

- Profile photo upload
- Document upload
- Support ticket attachments
- Video/audio upload metadata
- Mixed text + file form submission

## Multipart Structure

Multipart request body contains boundaries.

```text
--boundary
Content-Disposition: form-data; name="name"

Ashish
--boundary
Content-Disposition: form-data; name="avatar"; filename="avatar.jpg"
Content-Type: image/jpeg

<binary data>
--boundary--
```

## Multipart Builder

```swift
struct MultipartFormData {
    let boundary: String
    private(set) var data = Data()

    init(boundary: String = UUID().uuidString) {
        self.boundary = boundary
    }

    mutating func addField(name: String, value: String) {
        data.append("--\(boundary)\r\n".data(using: .utf8)!)
        data.append("Content-Disposition: form-data; name=\"\(name)\"\r\n\r\n".data(using: .utf8)!)
        data.append("\(value)\r\n".data(using: .utf8)!)
    }

    mutating func addFile(
        name: String,
        filename: String,
        contentType: String,
        fileData: Data
    ) {
        data.append("--\(boundary)\r\n".data(using: .utf8)!)
        data.append("Content-Disposition: form-data; name=\"\(name)\"; filename=\"\(filename)\"\r\n".data(using: .utf8)!)
        data.append("Content-Type: \(contentType)\r\n\r\n".data(using: .utf8)!)
        data.append(fileData)
        data.append("\r\n".data(using: .utf8)!)
    }

    mutating func finalize() {
        data.append("--\(boundary)--\r\n".data(using: .utf8)!)
    }
}
```

For production, avoid force unwraps and consider streaming/file upload for large files.

## Upload Request

```swift
func uploadAvatar(imageData: Data, displayName: String) async throws -> ProfileResponse {
    var multipart = MultipartFormData()
    multipart.addField(name: "display_name", value: displayName)
    multipart.addFile(
        name: "avatar",
        filename: "avatar.jpg",
        contentType: "image/jpeg",
        fileData: imageData
    )
    multipart.finalize()

    var request = URLRequest(url: baseURL.appending(path: "profile/avatar"))
    request.httpMethod = "POST"
    request.setValue(
        "multipart/form-data; boundary=\(multipart.boundary)",
        forHTTPHeaderField: "Content-Type"
    )

    let (data, response) = try await session.upload(for: request, from: multipart.data)
    return try decoder.decode(ProfileResponse.self, from: try validate(data, response))
}
```

## Upload From File

For large files, prefer uploading from a file URL.

```swift
let (data, response) = try await session.upload(for: request, fromFile: fileURL)
```

This avoids keeping the entire file in memory.

## Background Uploads

Use background URLSession for long uploads that should continue while app is suspended.

```swift
let configuration = URLSessionConfiguration.background(withIdentifier: "com.example.uploads")
let session = URLSession(configuration: configuration, delegate: delegate, delegateQueue: nil)
```

Background uploads generally work with file-based uploads, not arbitrary in-memory data.

## Progress

For progress UI, use delegate methods or task progress.

```swift
let task = session.uploadTask(with: request, fromFile: fileURL)
let progress = task.progress
task.resume()
```

In SwiftUI, bridge progress through observable state.

## Senior iOS Engineer Perspective

Ask:

- Is file small enough for in-memory upload?
- Should upload continue in background?
- Is content type correct?
- Are filenames safe?
- Is progress needed?
- Can upload be retried safely?
- Is authentication included?
- Is image compressed/resized before upload?

## Common Mistakes

- Loading huge video into memory.
- Missing boundary in Content-Type.
- Wrong line endings.
- No upload progress.
- Retrying upload without idempotency or resumability.
- No background session for long uploads.
- Uploading full-resolution images unnecessarily.

## Interview Notes

Junior:

Multipart upload sends fields and files in one request.

Mid-level:

Build `multipart/form-data` with boundaries and use URLSession upload APIs.

Senior:

I design uploads around memory, file-based transfer, background behavior, progress, retry safety, compression, authentication, and server contract.

## Practice

1. Build multipart body with text and image.
2. Upload from file instead of memory.
3. Add progress reporting.
4. Explain background upload constraints.
