# Day 13: Generics Interview Examples

## Example 1: Generic Decode

```swift
func decode<T: Decodable>(_ type: T.Type, from data: Data) throws -> T {
    try JSONDecoder().decode(T.self, from: data)
}
```

## Example 2: Generic Result View State

```swift
enum LoadState<Value> {
    case idle
    case loading
    case loaded(Value)
    case failed(Error)
}
```

## Example 3: Generic Cache

```swift
struct Cache<Key: Hashable, Value> {
    private var storage: [Key: Value] = [:]

    subscript(key: Key) -> Value? {
        get { storage[key] }
        set { storage[key] = newValue }
    }
}
```

## Interview Levels

Junior: Generics avoid writing the same code many times.

Senior: Generics are compile-time abstraction. They keep APIs reusable without sacrificing type safety.

## Modern Swift 6.x Notes

Swift 6.3 adds performance-control attributes for library authors such as specialization controls. Normal app developers mostly need to understand generic constraints, not micro-optimization attributes.

## Interview Depth

Junior answer: Generics help write reusable code without losing type information.

Mid-level answer: Common interview examples include generic decoding, generic state containers, generic caches, and generic repository helpers.

Senior answer: In real code, generics should improve correctness and reduce duplication. If a generic abstraction makes call sites harder to understand, a concrete type may be better.

iOS use case:

```swift
struct PagedResponse<Item: Decodable>: Decodable {
    let items: [Item]
    let nextPage: Int?
}
```

Common mistakes: generic for its own sake, replacing clear domain names with `T` everywhere, overusing type erasure.

Practice: explain `APIResponse<User>`, build generic cache, compare generic vs protocol.

## Deep Dive Expansion

### Why This Topic Matters In Real Apps

Generics Interview Examples is not only a syntax topic. In production Swift, it affects type-safe reuse, constraints, abstraction cost, and preserving static information. A junior developer should be able to recognize the syntax and explain the basic behavior. A mid-level developer should know when the feature improves clarity or safety. A senior developer should understand how it changes API design, testability, performance, or long-term maintenance.

In iOS work, this usually appears while network clients, repositories, reusable UI models, and type-erased dependencies. The important question is not "Can I write the syntax?" The important question is "Does this code make invalid states harder, make ownership clearer, and make future changes safer?"

### Beginner To Senior Progression

Beginner level:

- Define the concept in plain language.
- Write the smallest working example.
- Recognize the compiler error when the feature is used incorrectly.
- Explain what happens at the call site.

Mid-level level:

- Choose this feature over a nearby alternative for a clear reason.
- Handle edge cases, nil/error/cancellation/performance concerns where relevant.
- Keep the code readable for the next developer.
- Write tests around the behavior, not just the implementation detail.

Senior level:

- Explain the design tradeoff and the failure mode it prevents.
- Understand how this feature behaves across module, actor, memory, or API boundaries.
- Design examples that scale from a small screen to a larger feature.
- Avoid exposing implementation details as permanent API.

### Production-Style Example Pattern

Use this checklist when applying Generics Interview Examples in an app feature:

1. Identify the owner of the data or behavior.
2. Decide whether the value should be mutable, immutable, optional, throwing, async, isolated, or private.
3. Keep the public surface small and intention-revealing.
4. Add one realistic failure path, not only the happy path.
5. Check whether the code is still understandable from the call site.

```swift
struct FeatureState: Equatable {
    var isLoading: Bool
    var message: String?
    var canRetry: Bool
}

func makeInitialState() -> FeatureState {
    FeatureState(isLoading: false, message: nil, canRetry: false)
}
```

This small pattern is intentionally simple: define the state, control mutation through a narrow function, and make the result easy to inspect in tests.

### Edge Cases To Think About

- What happens when the input is empty, nil, duplicated, delayed, or invalid?
- What happens when this code is called repeatedly from a scrolling list or fast-changing UI?
- Does this API expose too much mutable state?
- Does the implementation assume a specific ordering, lifetime, actor, or thread?
- Will this still be easy to test after the feature grows?

### Topic-Specific Senior Notes

- Turn each answer into a story: situation, decision, tradeoff, result, and verification.
- For senior interviews, avoid only definitions. Explain why the design prevents bugs in a real app.
- Use generics when the caller benefits from preserving concrete type information.
- Reach for type erasure only when storage or heterogeneous collections require it.

### Common Interview Follow-Ups

Be ready for these follow-ups:

- Why did you choose this approach instead of the simpler alternative?
- What bug could happen if this is implemented carelessly?
- How would this behave in a large codebase with multiple modules?
- How would you test this without relying on UI screenshots?
- What changes when this code becomes async, public, generic, or shared?

### Strong Interview Framing

A strong answer should sound like this:

```text
I understand the basic syntax, but I also think about ownership and boundaries. I choose the approach that keeps state valid, makes the call site clear, and avoids unnecessary coupling. In a production iOS app, I would also consider testing, cancellation or error behavior where relevant, and whether this should remain an implementation detail or become part of a public API.
```

### Extra Practice

1. Write a minimal example that shows Generics Interview Examples correctly.
2. Write a second example that demonstrates a common mistake.
3. Refactor the mistake into a safer version.
4. Explain the difference between the beginner solution and the production-ready solution.
5. Turn one answer into a real project story with tradeoffs and verification.

## Applied Day-Level Example

The following example is not the only way to use Generics Interview Examples, but it shows the kind of production shape you should connect this topic to:

```swift
struct Endpoint<Response: Decodable> {
    let path: String
    let decode: (Data) throws -> Response
}

func send<Response>(_ endpoint: Endpoint<Response>) async throws -> Response {
    let data = try await loadData(path: endpoint.path)
    return try endpoint.decode(data)
}
```

Generics let the compiler preserve the response type from the endpoint to the call site, removing casts and reducing runtime mistakes.

### How To Study This Example

- Identify which part is syntax and which part is design.
- Ask what invalid state the example prevents.
- Ask what would need to change if the feature became async, generic, public, or shared across modules.
- Rewrite the example with one deliberate bug, then explain how a reviewer should catch it.
- Turn the example into a two-minute interview explanation with tradeoffs.

