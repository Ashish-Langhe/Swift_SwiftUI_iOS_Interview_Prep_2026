# Day 34: Download Tasks

## What Download Tasks Are

Download tasks retrieve content and save it to a temporary file URL.

Use downloads for:

- PDFs
- Videos
- Audio
- Large images
- Offline packages
- Export files
- Background transfers

For small JSON API responses, use data tasks. For large files, use download tasks.

## Basic Async Download

```swift
func downloadInvoice(id: Invoice.ID) async throws -> URL {
    let url = baseURL.appending(path: "invoices/\(id.rawValue)/pdf")
    let (temporaryURL, response) = try await session.download(from: url)

    guard let http = response as? HTTPURLResponse,
          200..<300 ~= http.statusCode else {
        throw APIError.invalidResponse
    }

    return try moveToDocuments(temporaryURL, filename: "invoice-\(id.rawValue).pdf")
}
```

URLSession gives a temporary file. Move it before returning if you need to keep it.

## Moving File

```swift
func moveToDocuments(_ temporaryURL: URL, filename: String) throws -> URL {
    let documents = FileManager.default.urls(
        for: .documentDirectory,
        in: .userDomainMask
    )[0]

    let destination = documents.appendingPathComponent(filename)

    if FileManager.default.fileExists(atPath: destination.path) {
        try FileManager.default.removeItem(at: destination)
    }

    try FileManager.default.moveItem(at: temporaryURL, to: destination)
    return destination
}
```

## Download With Request

```swift
var request = URLRequest(url: fileURL)
request.setValue("Bearer \(token)", forHTTPHeaderField: "Authorization")

let (temporaryURL, response) = try await session.download(for: request)
```

Use request-based download when headers/auth are needed.

## Resume Downloads

Some downloads can resume with resume data.

```swift
func urlSession(
    _ session: URLSession,
    task: URLSessionTask,
    didCompleteWithError error: Error?
) {
    let resumeData = (error as NSError?)?.userInfo[NSURLSessionDownloadTaskResumeData] as? Data
}
```

Resume:

```swift
let (url, response) = try await session.download(resumeFrom: resumeData)
```

Resume support depends on server and response headers.

## Background Downloads

```swift
let configuration = URLSessionConfiguration.background(withIdentifier: "com.example.downloads")
let session = URLSession(configuration: configuration, delegate: delegate, delegateQueue: nil)
```

Use for long downloads that should continue when app is suspended.

## Progress UI

Use `URLSessionDownloadDelegate`.

```swift
func urlSession(
    _ session: URLSession,
    downloadTask: URLSessionDownloadTask,
    didWriteData bytesWritten: Int64,
    totalBytesWritten: Int64,
    totalBytesExpectedToWrite: Int64
) {
    let progress = Double(totalBytesWritten) / Double(totalBytesExpectedToWrite)
}
```

Handle unknown expected length.

## Cache vs Documents

Store in Documents/Application Support if user/app needs it long-term.

Store in Caches if file can be recreated.

```text
Invoice PDF user saved:
Documents

Downloaded image thumbnail:
Caches

Offline learning video:
Application Support or Caches depending product promise
```

## Senior iOS Engineer Perspective

Ask:

- Is this small data or large file?
- Should it continue in background?
- Should it be cached or persisted?
- Is resume support needed?
- Is progress UI needed?
- Should file be excluded from backup?
- How is disk cleanup handled?

## Common Mistakes

- Using data tasks for huge files.
- Forgetting temporary download file must be moved.
- Storing cache files in Documents.
- No progress for long downloads.
- Assuming resume always works.
- No disk cleanup.
- No background session for long transfers.

## Interview Notes

Junior:

Download tasks save downloaded data to a temporary file.

Mid-level:

Move downloaded files to the right app directory and use delegates for progress.

Senior:

I design downloads around file size, background transfer, progress, resume behavior, disk location, backup policy, cleanup, and authentication.

## Practice

1. Download a PDF and move it to Documents.
2. Add progress delegate.
3. Design offline video storage.
4. Explain data task vs download task.
