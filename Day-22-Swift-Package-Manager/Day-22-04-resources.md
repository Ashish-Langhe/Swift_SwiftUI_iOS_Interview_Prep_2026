# Day 22: Resources

## What Resources Are

Swift packages can include resources such as:

- JSON files
- Images
- Localized strings
- Test fixtures
- Configuration files
- Asset catalogs

Resources are declared in a target.

```swift
.target(
    name: "ProfileFeature",
    resources: [
        .process("Resources"),
    ]
)
```

## Resource Layout

```text
Sources/
  ProfileFeature/
    ProfileView.swift
    Resources/
      profile-placeholder.json
      Images.xcassets
```

Resources belong to a target, not the whole package.

## `.process` vs `.copy`

```swift
resources: [
    .process("Resources"),
    .copy("RawFiles"),
]
```

Use `.process` for resources SwiftPM can process, optimize, or localize.

Use `.copy` when files must be preserved exactly.

## Accessing Resources

SPM generates `Bundle.module`.

```swift
let url = Bundle.module.url(
    forResource: "profile-placeholder",
    withExtension: "json"
)
```

Use `Bundle.module`, not `Bundle.main`, inside packages.

## Real iOS Use Case: Test Fixtures

```text
Tests/
  NetworkingTests/
    Resources/
      user-success.json
      user-missing-name.json
```

Manifest:

```swift
.testTarget(
    name: "NetworkingTests",
    dependencies: ["Networking"],
    resources: [
        .process("Resources"),
    ]
)
```

Test:

```swift
let url = Bundle.module.url(forResource: "user-success", withExtension: "json")!
let data = try Data(contentsOf: url)
```

## Localization

Packages can include localized resources.

Senior point:

If a reusable UI package owns user-visible text, it also owns localization strategy.

## Senior iOS Engineer Artifact

```text
Artifact: Package Resource Map
Target:
Resource type:
Process or copy:
Access API:
Localization needed:
Test fixture coverage:
Runtime failure handling:
Bundle.module usage verified:
```

Senior lens:

- Keep resources close to the target that owns them.
- Use `Bundle.module` from package code.
- Include invalid fixture files for tests.
- Treat localization as part of package API when UI is reusable.

## More Coding Examples

### Example 1: Load JSON Fixture

```swift
func fixtureData(named name: String) throws -> Data {
    guard let url = Bundle.module.url(forResource: name, withExtension: "json") else {
        throw FixtureError.missing(name)
    }
    return try Data(contentsOf: url)
}
```

### Example 2: Decode Fixture

```swift
let data = try fixtureData(named: "profile-success")
let response = try JSONDecoder().decode(ProfileResponse.self, from: data)
```

## Common Mistakes

- Using `Bundle.main` in package code.
- Forgetting to declare resources in the target.
- Mixing test fixtures into production targets.
- Force-unwrapping fixture URLs without useful test failure messages.
- Copying resources that should be processed.

## Interview Guide

Junior:

Packages can include resources and access them with `Bundle.module`.

Mid-level:

Declare resources on the target using `.process` or `.copy`, and keep test fixtures in test targets.

Senior:

Resources are owned by targets. I design resource layout around ownership, localization, testing, and runtime failure handling.

## Practice

1. Add JSON fixtures to a test target.
2. Load a fixture using `Bundle.module`.
3. Decide when to use `.process` vs `.copy`.
4. Explain why package code should not use `Bundle.main`.
