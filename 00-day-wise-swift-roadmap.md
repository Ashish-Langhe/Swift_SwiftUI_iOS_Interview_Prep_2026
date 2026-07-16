# Day-Wise Swift Fundamentals Roadmap

This roadmap is designed for iOS interview preparation in 2026. It starts from Swift basics and gradually moves into production-grade iOS, SwiftUI, concurrency, architecture, testing, and system design.

## Day 1: Swift Basics

Topics:

- Variables and constants
- Type inference and type annotations
- Primitive data types
- Literals
- Operators and expressions
- String interpolation
- Comments and documentation comments
- Basic print/debug output
- Swift 6.1, 6.2, and 6.3 changes beginners should know

Goal:

Become comfortable reading and writing simple Swift programs and explaining why `let`, `var`, and strong typing matter.

## Day 2: Control Flow

Topics:

- `if`, `else if`, `else`
- `switch`
- Pattern matching basics
- `for-in`, `while`, `repeat-while`
- `where` clauses
- `guard`
- Early exits
- Interview-focused differences between `if`, `guard`, and `switch`

## Day 3: Functions

Topics:

- Function declaration
- Parameters and return values
- Argument labels
- Default parameters
- Variadic parameters
- `inout`
- Function types
- Nested functions
- Higher-order functions introduction

## Day 4: Optionals

Topics:

- Optional meaning and memory model intuition
- Optional binding
- Optional chaining
- Nil-coalescing
- Force unwrap risks
- Implicitly unwrapped optionals
- Optional pattern matching
- Real iOS examples with API responses and view state

## Day 5: Collections

Topics:

- Arrays
- Dictionaries
- Sets
- Collection mutability
- Iteration patterns
- Sorting, filtering, mapping
- Collection performance basics
- When to use Array vs Set vs Dictionary

## Day 6: Strings And Characters

Topics:

- `String` vs `Character`
- Unicode and grapheme clusters
- String indexing
- Substrings
- Formatting
- Localization basics
- Common interview traps

## Day 7: Tuples, Ranges, And Pattern Matching

Topics:

- Tuples
- Named tuple elements
- Returning multiple values
- Ranges
- Pattern matching in `switch`
- `if case`, `guard case`, `for case`

## Day 8: Structs And Value Types

Topics:

- Struct declaration
- Stored properties
- Computed properties
- Initializers
- Methods
- Mutating methods
- Value semantics
- Copy-on-write intuition
- Why Swift prefers structs

## Day 9: Classes And Reference Types

Topics:

- Class declaration
- Reference semantics
- Identity vs equality
- Initializers and deinitializers
- Inheritance
- Method overriding
- `final`
- Memory implications

## Day 10: Enums

Topics:

- Basic enums
- Associated values
- Raw values
- Recursive enums
- CaseIterable
- Identifiable enums
- State modeling in iOS
- Enum-driven UI

## Day 11: Properties

Topics:

- Stored properties
- Computed properties
- Lazy properties
- Property observers
- Type properties
- Access control on properties
- Property wrappers introduction

## Day 12: Protocols

Topics:

- Protocol declaration
- Requirements
- Protocol conformance
- Protocol extensions
- Associated types
- Existentials with `any`
- Opaque types with `some`
- Protocol-oriented programming

## Day 13: Generics

Topics:

- Generic functions
- Generic types
- Type constraints
- Associated types with generics
- Generic where clauses
- Type erasure introduction
- Interview examples

## Day 14: Error Handling

Topics:

- `Error` protocol
- `throw`, `throws`, `try`
- `try?`, `try!`
- `do-catch`
- Typed error modeling
- Result type
- Error handling in async code

## Day 15: Closures

Topics:

- Closure syntax
- Capturing values
- Escaping vs non-escaping closures
- Autoclosures
- Retain cycles
- Capture lists
- Closures in SwiftUI and UIKit

## Day 16: Memory Management

Topics:

- ARC
- Strong references
- Weak references
- Unowned references
- Retain cycles
- Closures and memory leaks
- Debugging memory issues

## Day 17: Access Control And Modules

Topics:

- `private`
- `fileprivate`
- `internal`
- `package`
- `public`
- `open`
- Module boundaries
- Framework design basics

## Day 18: Concurrency Basics

Topics:

- `async` and `await`
- Tasks
- Structured concurrency
- Task cancellation
- Task priority
- Async sequences
- Main actor basics

## Day 19: Modern Swift Concurrency

Topics:

- Actors
- Global actors
- `Sendable`
- Strict concurrency
- Data-race safety
- Swift 6 migration issues
- Swift 6.2 approachable concurrency
- `@concurrent`
- Default actor isolation

## Day 20: Advanced Concurrency And Networking

Topics:

- URLSession async APIs
- Parallel async work
- Task groups
- Async let
- AsyncStream
- Retry and timeout patterns
- Actor-isolated services

## Day 21: Swift Standard Library Deep Dive

Topics:

- Collection protocols
- Sequences
- Lazy sequences
- Result builders
- Key paths
- Codable
- Comparable, Hashable, Equatable

## Day 22: Swift Package Manager

Topics:

- Package structure
- Targets and products
- Dependencies
- Resources
- Plugins
- Package traits from Swift 6.1
- Warning control from Swift 6.2

## Day 23: Macros And Code Generation

Topics:

- Macro purpose
- Freestanding macros
- Attached macros
- SwiftSyntax basics
- Macro use cases
- Build performance improvements from Swift 6.2

## Day 24: Swift Testing

Topics:

- Swift Testing basics
- `@Test`
- `#expect`
- Traits
- Custom test scoping from Swift 6.1
- Exit testing from Swift 6.2
- Attachments from Swift 6.2

## Day 25: SwiftUI Fundamentals

Topics:

- View protocol
- State
- Binding
- Observable state
- Environment
- View identity
- Layout basics
- Navigation

## Day 26: SwiftUI Intermediate

Topics:

- Lists
- Forms
- Sheets
- Alerts
- Animations
- Custom components
- MVVM in SwiftUI
- Observation framework

## Day 27: iOS App Architecture

Topics:

- MVC, MVVM, VIPER, TCA overview
- Dependency injection
- Coordinator pattern
- Repository pattern
- Service layers
- State management
- Testability

## Day 28: Persistence

Topics:

- UserDefaults
- Keychain
- FileManager
- Codable persistence
- SQLite basics
- SwiftData/Core Data overview
- Migration basics

## Day 29: Performance And Debugging

Topics:

- Instruments
- Time Profiler
- Memory graph
- Allocations
- SwiftUI performance
- Main-thread work
- Copy-on-write performance
- Async debugging improvements from Swift 6.2

## Day 30: Interview Revision And Projects

Topics:

- Common Swift interview questions
- Common iOS interview questions
- Coding exercises
- Mini project ideas
- Debugging challenges
- System design for iOS apps
- Final revision checklist

