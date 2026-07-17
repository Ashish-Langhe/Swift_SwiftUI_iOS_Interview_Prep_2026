# Day 21: Swift Standard Library Interview Guide

## One-Minute Senior Answer

Swift's standard library is not just utility syntax. It defines the contracts behind collections, iteration, transformation, equality, hashing, ordering, encoding, and declarative builders. A senior iOS engineer uses these tools to design APIs that are type-safe, performant, readable, and testable. I choose protocol constraints based on real algorithm needs, use lazy evaluation only when it helps, model Codable boundaries carefully, and define equality/hash/order according to domain identity.

## Core Mental Models

- `Sequence`: one-pass iteration.
- `Collection`: stable indexing and repeated traversal.
- `RandomAccessCollection`: efficient offset movement.
- Lazy sequences: delayed transformation and fewer intermediate allocations.
- Result builders: declarative block-to-value transformation.
- Key paths: type-safe property access as a value.
- Codable: serialization boundary, not always domain modeling.
- Equatable/Hashable/Comparable: equality, identity, hashing, and ordering contracts.

## Junior Questions

What is a sequence?

Something you can iterate with `for-in`.

What is Codable?

A protocol combination that lets types encode and decode data.

What is Equatable?

A protocol that lets values be compared with `==`.

## Mid-Level Questions

Sequence vs Collection?

`Sequence` is for iteration. `Collection` supports stable indices, count, and repeated traversal.

When use lazy?

When avoiding intermediate arrays or stopping early matters enough to justify the readability cost.

What are key paths good for?

Mapping, sorting, generic property access, and simple property-based APIs.

## Senior Questions

How do you choose collection protocol constraints?

I start with the minimum capability the algorithm needs. If I only iterate once, `Sequence` may be enough. If I need count or indexing, I use `Collection`. If offset access must be efficient, I require `RandomAccessCollection`.

How do you design Codable models?

I separate transport DTOs from domain models when the API shape is weaker than the app's invariants. I test invalid payloads, optional behavior, dates, and key strategies.

How do you decide equality?

I ask whether equality means full value equality or stable identity. That decision affects sets, dictionaries, SwiftUI identity, diffing, and tests.

## Senior iOS Engineer Artifact

```text
Artifact: Standard Library Design Review
Feature: What screen/service is being built?
Data shape: Sequence, Collection, Array, Set, Dictionary, custom type
Transformation: eager, lazy, sorted, grouped, reduced
Serialization: Codable DTO or domain model?
Identity: Equatable, Hashable, Identifiable, Comparable
API boundary: concrete type or protocol constraint?
Performance risk: allocation, indexing cost, repeated scans, sorting cost
Test evidence: edge case, invalid payload, identity behavior, ordering behavior
```

## Real iOS Scenario

You are building a product list.

Strong design:

```swift
struct ProductResponse: Decodable {
    let id: String
    let title: String
    let price: Decimal
}

struct Product: Identifiable, Hashable {
    let id: String
    let title: String
    let price: Decimal
}

func visibleProducts(
    from products: [Product],
    selectedIDs: Set<Product.ID>,
    query: String
) -> [ProductRowViewModel] {
    products.lazy
        .filter { query.isEmpty || $0.title.localizedCaseInsensitiveContains(query) }
        .sorted { $0.title < $1.title }
        .map {
            ProductRowViewModel(
                id: $0.id,
                title: $0.title,
                isSelected: selectedIDs.contains($0.id)
            )
        }
}
```

Senior discussion:

- Codable response is a transport model.
- Product has stable identity.
- Set gives fast selected lookup.
- Lazy avoids intermediate arrays before sort, though sort materializes.
- Row model is UI-specific.
- Search and sort rules should be tested.

## Common Traps

- Using `Array` constraints everywhere.
- Assuming all sequences are reusable.
- Using lazy without understanding evaluation timing.
- Creating result builders that make errors harder.
- Making everything Codable and using DTOs as domain models.
- Letting synthesized Hashable define the wrong identity.
- Sorting with unclear business meaning.

## Interview Closing Answer

At senior level, I treat standard-library protocols as design contracts. They let me express exactly what an algorithm needs, reduce unnecessary concrete coupling, and keep data transformations type-safe. But I also watch for performance, readability, serialization boundaries, and identity semantics because those choices affect real iOS behavior.

## Practice Prompts

1. Design an API that accepts the weakest useful collection protocol.
2. Use lazy to find the first matching item in a large list.
3. Build a small result builder for validation rules.
4. Use key paths to sort users.
5. Decode a DTO and map it into a stronger domain model.
6. Define equality and hashing for a diffable UI row.
