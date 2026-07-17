# Day 32: Copy-On-Write Performance

## What Copy-On-Write Means

Swift collections like `Array`, `Dictionary`, `Set`, and `String` are value types with copy-on-write behavior.

This means copying a value is cheap until one copy is mutated.

```swift
var a = [1, 2, 3]
var b = a       // cheap shared storage
b.append(4)    // storage may copy here
```

## Why COW Matters For Performance

COW gives value semantics without copying every time.

But performance issues happen when:

- Large collections are copied and mutated repeatedly.
- Mutations happen inside loops on shared storage.
- Values are passed through many layers and modified.
- Large structs contain large collections.
- SwiftUI state repeatedly copies large values.

## Example: Repeated Array Mutation

Potentially expensive:

```swift
func appendItems(_ items: [Item], to existing: [Item]) -> [Item] {
    var result = existing

    for item in items {
        result.append(item)
    }

    return result
}
```

Usually okay if storage is uniquely referenced. But if not, mutation triggers copy.

You can reserve capacity:

```swift
var result = existing
result.reserveCapacity(existing.count + items.count)
result.append(contentsOf: items)
```

## Large Value In SwiftUI State

Risky:

```swift
@State private var document = LargeDocument()
```

Every mutation to nested large arrays may cause expensive copies depending on structure and sharing.

Consider:

- Smaller state slices
- Reference-backed model
- Persistent storage boundary
- Editing draft model
- Incremental updates

## Dictionary COW

```swift
var cache = [Product.ID: Product]()

for product in products {
    cache[product.id] = product
}
```

This is normally efficient if `cache` is uniquely referenced. Problems arise when large dictionaries are copied around and mutated in multiple places.

## Checking Uniqueness In Custom COW

Custom COW uses reference storage.

```swift
final class Buffer {
    var values: [Int]

    init(values: [Int]) {
        self.values = values
    }
}

struct CowArray {
    private var buffer: Buffer

    init(values: [Int]) {
        buffer = Buffer(values: values)
    }

    mutating func append(_ value: Int) {
        if !isKnownUniquelyReferenced(&buffer) {
            buffer = Buffer(values: buffer.values)
        }

        buffer.values.append(value)
    }
}
```

This is advanced, but useful for understanding how COW works.

## Measuring COW

Use:

- Time Profiler for CPU cost.
- Allocations for copy allocations.
- Benchmarks for isolated operations.

Look for:

- Array copies
- Large allocations during mutation
- Unexpected growth in value transformations

## Common Mistakes

- Assuming value types always copy immediately.
- Assuming COW means copying is never expensive.
- Mutating huge value state repeatedly in SwiftUI.
- Large arrays in frequently updated observable state.
- No `reserveCapacity` for known growth.
- Optimizing COW without measurement.

## Senior Artifact

```text
Artifact: Copy-On-Write Performance Review
Type:
Collection size:
Mutation frequency:
Shared storage risk:
Allocation trace:
CPU trace:
Reserve capacity:
State slicing:
Fix:
Verification:
```

## Interview Notes

Junior:

Swift arrays are value types, but they use copy-on-write to avoid unnecessary copies.

Mid-level:

Copies become expensive when large shared storage is mutated.

Senior:

I understand COW as a value-semantics optimization, measure allocation/CPU cost before changing design, and avoid repeatedly mutating huge value state in hot UI paths.

## Practice

1. Explain when array copy actually happens.
2. Add `reserveCapacity` before appending many items.
3. Profile large value mutations.
4. Implement a tiny custom COW wrapper.
