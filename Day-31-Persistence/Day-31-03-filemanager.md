# Day 31: FileManager

## What FileManager Is

`FileManager` is Foundation's interface for working with the file system.

Use it to:

- Locate app directories
- Create folders
- Read/write files
- Move/copy/delete files
- Inspect file metadata
- Manage cache files

```swift
let documents = FileManager.default.urls(
    for: .documentDirectory,
    in: .userDomainMask
)[0]
```

Apple recommends using file URLs rather than plain string paths.

## App Sandbox Directories

Common directories:

- Documents: user-created or important app data.
- Library/Application Support: app support files.
- Library/Caches: cached data that can be recreated.
- tmp: temporary files.

```swift
let support = try FileManager.default.url(
    for: .applicationSupportDirectory,
    in: .userDomainMask,
    appropriateFor: nil,
    create: true
)
```

## Writing Data

```swift
struct FileStore {
    let directory: URL

    func save(_ data: Data, filename: String) throws {
        try FileManager.default.createDirectory(
            at: directory,
            withIntermediateDirectories: true
        )

        let url = directory.appendingPathComponent(filename)
        try data.write(to: url, options: [.atomic])
    }
}
```

Use `.atomic` to reduce risk of partially written files.

## Reading Data

```swift
func load(filename: String) throws -> Data {
    let url = directory.appendingPathComponent(filename)
    return try Data(contentsOf: url)
}
```

For large files, avoid loading everything into memory. Use streaming APIs.

## Deleting

```swift
func delete(filename: String) throws {
    let url = directory.appendingPathComponent(filename)

    if FileManager.default.fileExists(atPath: url.path) {
        try FileManager.default.removeItem(at: url)
    }
}
```

## Cache Directory

```swift
let cacheDirectory = FileManager.default.urls(
    for: .cachesDirectory,
    in: .userDomainMask
)[0]
```

Use cache for data that can be regenerated.

Examples:

- Image cache
- Downloaded thumbnails
- Temporary API response cache
- Rendered previews

Do not store user-created irreplaceable data in Caches.

## Backup Considerations

Documents and Application Support may be backed up. Caches and tmp are not intended for permanent data.

For large non-user-generated files that should not back up, set resource values carefully.

```swift
var values = URLResourceValues()
values.isExcludedFromBackup = true

var url = fileURL
try url.setResourceValues(values)
```

## File Names

Avoid unsafe filenames from user input.

```swift
func safeFilename(for id: String) -> String {
    id.replacingOccurrences(of: "/", with: "_")
}
```

Better: use stable IDs or UUID-based filenames.

## FileStore Protocol

```swift
protocol FileStoring {
    func save(_ data: Data, name: String) throws
    func load(name: String) throws -> Data
    func delete(name: String) throws
}
```

Use protocol abstraction for tests and feature boundaries.

## Common Mistakes

- Using string paths instead of URLs.
- Storing cache data in Documents.
- Storing important data in tmp.
- Not creating directories before writing.
- No atomic writes.
- Loading huge files fully into memory.
- Not excluding large recreated files from backup.

## Senior iOS Engineer Perspective

Ask:

- Is this user data, app support data, cache, or temporary?
- Should it be backed up?
- Is the write atomic?
- Can file names collide?
- Is the data too large for memory loading?
- How will cleanup happen?

## Interview Notes

Junior:

`FileManager` reads and writes files in app directories.

Mid-level:

Choose the right sandbox directory and use URLs with atomic writes.

Senior:

I design file persistence around data lifetime, backup policy, atomicity, memory use, cleanup, and testability through file-store abstractions.

## Practice

1. Save JSON data to Application Support.
2. Cache an image in Caches.
3. Exclude a recreated file from backup.
4. Explain Documents vs Caches vs tmp.
