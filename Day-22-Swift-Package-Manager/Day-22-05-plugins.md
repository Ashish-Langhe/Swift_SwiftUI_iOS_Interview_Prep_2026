# Day 22: Plugins

## What SwiftPM Plugins Are

SwiftPM plugins automate package-related tasks.

They can:

- Generate code
- Run linters
- Format code
- Generate resources
- Integrate tools into package workflows
- Add build tool commands

Plugins are targets in a package.

```swift
.plugin(
    name: "GenerateMocksPlugin",
    capability: .command(
        intent: .custom(verb: "generate-mocks", description: "Generate mock types")
    )
)
```

## Plugin Types

Common capabilities:

- Command plugins
- Build tool plugins

Command plugins run when invoked.

Build tool plugins participate in the build.

## Basic Command Plugin Shape

```swift
import PackagePlugin

@main
struct GenerateMocksPlugin: CommandPlugin {
    func performCommand(
        context: PluginContext,
        arguments: [String]
    ) async throws {
        print("Generate mocks for package at \\(context.package.directory)")
    }
}
```

## Real iOS Use Cases

- Generate API clients from OpenAPI files.
- Generate mocks for protocols.
- Validate localization files.
- Run SwiftFormat or SwiftLint.
- Generate asset constants.
- Build feature flags from config.

## Build Tool Plugin Idea

A build tool plugin can generate source files before compilation.

Senior caution:

Build tool plugins affect build time, determinism, CI, and developer trust. Generated code should be predictable.

## Plugin Package Layout

```text
Package.swift
Plugins/
  GenerateMocksPlugin/
    GenerateMocksPlugin.swift
Sources/
  MockGenerator/
    main.swift
```

Often, the plugin invokes an executable target that does the actual work.

## Senior iOS Engineer Artifact

```text
Artifact: Plugin Design Review
Plugin purpose:
Command or build tool:
Inputs:
Outputs:
Generated files:
Deterministic? yes/no
CI behavior:
Failure messages:
Performance impact:
Security risk:
```

Senior lens:

- Plugins should solve repetitive work, not hide unclear process.
- Generated outputs must be deterministic.
- Build plugins should be fast and predictable.
- Error messages should help app developers fix problems.

## More Coding Examples

### Example 1: Plugin Target In Manifest

```swift
targets: [
    .plugin(
        name: "ValidateLocalizationPlugin",
        capability: .command(
            intent: .custom(
                verb: "validate-localization",
                description: "Validate localization keys"
            )
        )
    ),
]
```

### Example 2: Plugin Reads Package Directory

```swift
import PackagePlugin

@main
struct ValidateLocalizationPlugin: CommandPlugin {
    func performCommand(context: PluginContext, arguments: [String]) async throws {
        let packageDirectory = context.package.directory
        print("Validate resources in \\(packageDirectory)")
    }
}
```

## Common Mistakes

- Generating nondeterministic files.
- Hiding slow work in every build.
- Poor failure messages.
- Requiring local tools that CI does not have.
- Writing generated files into source directories unexpectedly.
- Using plugins when a simple script is enough.

## Interview Guide

Junior:

SwiftPM plugins automate commands or build steps.

Mid-level:

Use command plugins for manual tasks and build tool plugins for generated build inputs.

Senior:

Plugins are workflow infrastructure. I evaluate determinism, build impact, CI behavior, security, and developer experience before adding them.

## Practice

1. Define a command plugin in `Package.swift`.
2. Sketch a localization validation plugin.
3. Explain command vs build tool plugin.
4. Review whether a generator should be a plugin or CI script.
