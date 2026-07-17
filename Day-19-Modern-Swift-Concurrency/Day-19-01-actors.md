# Day 19: Actors

## What Actors Are

An actor is a reference type that protects its mutable state from concurrent access.

```swift
actor TokenStore {
    private var token: String?

    func update(_ token: String) {
        self.token = token
    }

    func currentToken() -> String? {
        token
    }
}
```

Only one piece of actor-isolated code runs on an actor at a time.

## Why Actors Exist

Data races happen when multiple concurrent operations access the same mutable state and at least one access is a write.

Actors solve this by serializing access to protected state.

```swift
let store = TokenStore()
await store.update("abc")
let token = await store.currentToken()
```

Calls from outside the actor usually require `await`.

## Actor Isolation

Actor-isolated state can be accessed directly from inside the actor.

```swift
actor Counter {
    private var value = 0

    func increment() {
        value += 1
    }
}
```

Outside code must cross the actor boundary:

```swift
let counter = Counter()
await counter.increment()
```

## Actors Are Reference Types

```swift
let a = Counter()
let b = a
```

`a` and `b` refer to the same actor instance.

Actor does not mean value type. It means reference type with isolated mutable state.

## Reentrancy

Actors can be reentrant at suspension points.

```swift
actor BankAccount {
    private var balance: Decimal = 100

    func withdraw(_ amount: Decimal) async -> Bool {
        guard balance >= amount else { return false }
        await audit()
        balance -= amount
        return true
    }
}
```

After `await audit()`, another call may have run and changed actor state. Senior engineers avoid assuming state is unchanged across suspension.

Better:

```swift
func withdraw(_ amount: Decimal) async -> Bool {
    guard balance >= amount else { return false }
    balance -= amount
    await audit()
    return true
}
```

Update critical state before suspension when appropriate.

## Real iOS Actor Use Cases

Use actors for:

- Auth token storage
- In-memory caches
- Download coordination
- Image loading deduplication
- File write coordination
- Analytics event buffers
- Rate limiters

```swift
actor ImageCache {
    private var images: [URL: UIImage] = [:]

    func image(for url: URL) -> UIImage? {
        images[url]
    }

    func insert(_ image: UIImage, for url: URL) {
        images[url] = image
    }
}
```

## Actor vs Class

Class:

```swift
final class Counter {
    var value = 0
}
```

Unsafe if accessed from multiple tasks.

Actor:

```swift
actor SafeCounter {
    private var value = 0
}
```

Compiler enforces isolated access.

## Common Mistakes

- Thinking actors make all code inside instantly parallel
- Holding invalid assumptions across `await`
- Using actors for everything, including simple value models
- Updating UI from a non-main actor
- Returning mutable non-Sendable state from actors
- Blocking inside an actor

## Modern Swift 6.x Notes

Swift 6 data-race safety makes actor isolation much more important. The compiler can diagnose unsafe access to shared mutable state and actor-isolated members.

Swift 6.2 approachable concurrency reduces annotation overhead in UI-style code, but actors remain the core tool for protecting shared mutable state outside UI.

## Junior Interview Answer

An actor is like a class that protects its mutable state so only one task accesses it at a time.

## Mid-Level Interview Answer

Actors isolate state. Calls from outside an actor usually require `await`, and the compiler prevents direct unsafe access to actor-isolated properties.

## Senior Interview Answer

Actors are isolation domains for mutable reference state. I use them for shared services and caches, design APIs that avoid leaking mutable state, and watch for reentrancy around suspension points. Actors improve data-race safety, but they do not remove the need for careful ordering and cancellation.

## Practice

1. Convert a class-based cache to an actor.
2. Explain why actor calls require `await`.
3. Find a reentrancy bug around an `await`.
4. Design an actor for auth token refresh coordination.
