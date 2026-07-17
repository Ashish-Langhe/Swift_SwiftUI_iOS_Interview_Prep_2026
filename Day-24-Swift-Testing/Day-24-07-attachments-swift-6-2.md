# Day 24: Attachments From Swift 6.2

## What Attachments Are

Swift 6.2 added attachments to Swift Testing.

Attachments let tests include extra diagnostic context such as:

- strings
- logs
- JSON payloads
- images
- screenshots
- generated files
- traces

They are surfaced in test reports or written to disk.

## Why Attachments Matter

When a test fails, the failure line may not be enough.

Attachments can show:

- the decoded JSON payload
- the rendered UI screenshot
- the request/response body
- the state transition trace
- an image diff

This helps debug failures faster.

## Real iOS Use Case

Snapshot-like UI verification:

```swift
@Test
func profileHeader_rendersExpectedState() async throws {
    let image = try await renderProfileHeader()
    // Attach rendered image for failure diagnosis.
}
```

Even if the assertion fails, the test report can include visual evidence.

## API Payload Debugging

```swift
@Test
func userResponse_decodes() throws {
    let data = try fixtureData(named: "user-response")
    // Attach raw JSON when decoding fails.
    let user = try JSONDecoder().decode(UserResponse.self, from: data)
    #expect(user.id == "1")
}
```

Attachments make fixture failures easier to understand.

## Senior iOS Engineer Artifact

```text
Artifact: Test Attachment Plan
Test:
Attachment type:
When useful:
Size risk:
Privacy risk:
CI/reporting behavior:
Retention expectation:
Debug value:
```

Senior lens:

- Attachments are evidence.
- Avoid attaching secrets or huge files.
- Use attachments where failures are otherwise hard to diagnose.
- Screenshots, JSON, and logs are high-value artifacts.

## More Coding Examples

### Example 1: Attach JSON Conceptually

```swift
@Test
func profileJSON_decodes() throws {
    let data = try fixtureData(named: "profile")
    let json = String(decoding: data, as: UTF8.self)
    // Attachment.record("profile.json", json)
    #expect(!json.isEmpty)
}
```

### Example 2: Attach State Trace Conceptually

```swift
var trace: [String] = []
trace.append("state=loading")
trace.append("state=loaded")
// Attachment.record("state-trace.txt", trace.joined(separator: "\\n"))
```

Exact attachment APIs may vary by toolchain, but the senior idea is to preserve useful debugging evidence.

## Common Mistakes

- Attaching sensitive data.
- Attaching huge files by default.
- Adding attachments to every test without value.
- Depending on attachments instead of clear expectations.
- Not checking CI report support.

## Interview Guide

Junior:

Attachments add extra files or data to test results.

Mid-level:

Use attachments for screenshots, JSON, logs, and traces when failures need context.

Senior:

Attachments are diagnostic artifacts. I design them with privacy, size, retention, and CI visibility in mind.

## Practice

1. Decide what to attach for a JSON decoding failure.
2. Decide what to attach for a UI rendering failure.
3. Identify privacy risks in test artifacts.
4. Add an attachment plan to a flaky test investigation.
