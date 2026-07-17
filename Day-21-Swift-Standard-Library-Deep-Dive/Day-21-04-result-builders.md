# Day 21: Result Builders

## What Result Builders Are

Result builders let Swift build a value from a block of expressions.

SwiftUI uses result builders heavily:

```swift
var body: some View {
    VStack {
        Text("Title")
        Text("Subtitle")
    }
}
```

The block is transformed by a result builder into a single value.

## Why They Matter

Result builders power declarative APIs.

Common places:

- SwiftUI `ViewBuilder`
- Custom DSLs
- HTML builders
- Form builders
- Validation builders
- Test data builders

## Simple Custom Builder

```swift
@resultBuilder
enum StringListBuilder {
    static func buildBlock(_ components: String...) -> [String] {
        components
    }
}

func makeList(@StringListBuilder _ content: () -> [String]) -> [String] {
    content()
}

let items = makeList {
    "Profile"
    "Settings"
    "Logout"
}
```

The block produces `[String]`.

## Supporting Conditionals

```swift
@resultBuilder
enum MenuBuilder {
    static func buildBlock(_ components: [String]...) -> [String] {
        components.flatMap { $0 }
    }

    static func buildExpression(_ expression: String) -> [String] {
        [expression]
    }

    static func buildOptional(_ component: [String]?) -> [String] {
        component ?? []
    }
}
```

This allows:

```swift
let menu = makeMenu {
    "Home"
    if isLoggedIn {
        "Profile"
    }
}
```

## Real iOS Use Case

Custom form configuration:

```swift
struct FormField {
    let title: String
    let isRequired: Bool
}

@resultBuilder
enum FormBuilder {
    static func buildBlock(_ fields: FormField...) -> [FormField] {
        fields
    }
}

func makeForm(@FormBuilder _ fields: () -> [FormField]) -> [FormField] {
    fields()
}

let form = makeForm {
    FormField(title: "Email", isRequired: true)
    FormField(title: "Phone", isRequired: false)
}
```

## Senior iOS Engineer Artifact

```text
Artifact: Result Builder API Design
Topic: Result Builders
Problem: Does a declarative DSL improve readability?
Output type: What does the builder produce?
Control flow support: if/else, optional, arrays?
Debuggability: Are errors understandable?
Alternative: Would an array literal or initializer be clearer?
```

Senior lens:

- Do not create a builder just because SwiftUI uses them.
- Builders are best when declaration reads better than construction.
- Error messages can become harder for users of the API.
- Keep custom builders small and well documented.

## More Coding Examples

### Example 1: Validation Rules Builder

```swift
struct Rule {
    let message: String
    let validate: (String) -> Bool
}

@resultBuilder
enum RuleBuilder {
    static func buildBlock(_ rules: Rule...) -> [Rule] {
        rules
    }
}
```

### Example 2: Use The Builder

```swift
func makeRules(@RuleBuilder _ content: () -> [Rule]) -> [Rule] {
    content()
}

let rules = makeRules {
    Rule(message: "Required") { !$0.isEmpty }
    Rule(message: "At least 8 characters") { $0.count >= 8 }
}
```

## Common Mistakes

- Building a DSL where an array is simpler.
- Forgetting conditional support methods.
- Returning overly complex generic types.
- Hiding side effects in builder expressions.
- Making compile errors difficult for app developers.

## Interview Guide

Junior:

Result builders transform a block of expressions into a final value. SwiftUI uses them for view bodies.

Mid-level:

They are useful for declarative APIs where a block reads more naturally than manually appending values.

Senior:

Result builders are API design tools. I use them when they improve call-site clarity enough to justify their debugging and complexity costs.

## Practice

1. Build a simple string list builder.
2. Add conditional support.
3. Create a form field builder.
4. Explain when not to use a result builder.
