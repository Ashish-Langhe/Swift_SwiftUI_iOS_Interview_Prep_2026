# Day 16: ARC

## What ARC Means

ARC stands for Automatic Reference Counting. It is Swift's memory management system for class instances.

ARC automatically:

- Counts strong references to class instances
- Keeps an object alive while at least one strong reference exists
- Deallocates the object when no strong references remain

```swift
final class UserSession {
    let id: String

    init(id: String) {
        self.id = id
        print("Session created")
    }

    deinit {
        print("Session destroyed")
    }
}

var session: UserSession? = UserSession(id: "s1")
session = nil
```

When `session` becomes `nil`, the object has no strong references, so ARC can deallocate it.

## Beginner Mental Model

Think of every class instance as an object with an invisible ownership counter.

```text
UserSession object
strong reference count: 1
```

Every strong reference increases the count. Removing a strong reference decreases it. When the count reaches zero, ARC calls `deinit` and releases the object.

```swift
var first: UserSession? = UserSession(id: "s1") // count +1
var second = first                              // count +1

first = nil                                    // count -1
second = nil                                   // count -1, deinit can run
```

ARC is predictable because object lifetime follows references.

## ARC Is Not Garbage Collection

Garbage collection usually runs later and scans memory to find unused objects. ARC releases objects as soon as the strong reference count reaches zero.

Interview answer:

Swift ARC is deterministic reference counting, not tracing garbage collection. This gives predictable object lifetimes, but ARC cannot automatically solve strong reference cycles.

## ARC Applies To Classes

ARC manages reference types, mainly class instances.

Structs and enums are value types and are not reference-counted in the same way.

```swift
struct UserValue {
    let name: String
}

final class UserObject {
    let name: String

    init(name: String) {
        self.name = name
    }
}
```

`UserValue` is copied by value. `UserObject` is shared by reference and managed by ARC.

## Value Types Can Still Contain References

ARC does not manage value types like class instances, but value types can store references.

```swift
struct ImageRow {
    let title: String
    let loader: ImageLoader
}

final class ImageLoader { }
```

`ImageRow` is a value type, but `loader` is a class reference. Copying `ImageRow` copies the reference to the same loader object.

Senior interview point:

Value type does not always mean there is no reference behavior anywhere inside. You must inspect stored properties too.

## Real iOS Use Cases

ARC manages:

- `UIViewController`
- View models implemented as classes
- Services
- Coordinators
- Delegates
- Closures that capture class instances

```swift
final class ProfileViewModel {
    deinit {
        print("ProfileViewModel released")
    }
}
```

Adding `deinit` logs is a simple first step when checking if an object is leaking.

## ARC And Object Lifetimes In iOS

Typical healthy ownership chain:

```text
NavigationController
  strongly owns ViewController
    strongly owns ViewModel
      strongly owns Service
```

Problem ownership chain:

```text
ViewController
  strongly owns ViewModel
    strongly owns Closure
      strongly captures ViewController
```

This cycle prevents ARC from reaching a zero strong reference count.

## Advanced ARC Intuition

ARC inserts retain and release operations automatically. You normally do not write those operations yourself.

Important consequences:

- Passing class instances around can affect reference counts.
- Storing a class instance strongly keeps it alive.
- Capturing a class instance in an escaping closure keeps it alive.
- `weak` and `unowned` do not increase the strong reference count.

ARC is automatic, but ownership design is still your job.

## Modern Swift 6.x Notes

ARC is still the core memory management model for class instances. Modern Swift adds more safety around memory access and unsafe constructs:

- Swift 6.2 introduced opt-in strict memory safety diagnostics for unsafe APIs.
- Swift 6.2 introduced `Span` for safe access to contiguous memory.
- Swift 6 concurrency checking makes shared mutable reference state more visible and more constrained.

These do not replace ARC. They complement it.

## Junior-Level Interview Answer

ARC automatically manages memory for class instances by counting references. When no strong references remain, the object is deallocated.

## Mid-Level Interview Answer

ARC works with strong, weak, and unowned references. Strong references keep objects alive. Weak and unowned references do not. ARC prevents most manual memory management issues, but retain cycles can still leak memory.

## Senior-Level Interview Answer

ARC is deterministic reference-counting for object lifetimes. It is not a garbage collector. Objects are released when their strong reference count reaches zero. The main production risks are ownership mistakes: strong reference cycles, closure captures, global/shared references, and long-lived caches. In modern Swift, I also consider concurrency isolation because shared mutable class instances can create both lifetime and data-race problems.

## Common Mistakes

- Thinking ARC prevents all memory leaks.
- Forgetting closures capture strongly by default.
- Making delegates strong.
- Keeping objects alive through singleton caches.
- Assuming `deinit` will run while a strong reference still exists.

## Interview Progression

Basic:

ARC counts references and frees class instances when no strong references remain.

Intermediate:

Strong references keep objects alive. Weak and unowned references do not. Retain cycles can leak memory because reference counts never reach zero.

Advanced:

ARC gives deterministic object lifetime, but ownership graphs must be designed carefully. I verify lifetimes with `deinit`, inspect retain paths with Memory Graph, and pay special attention to closures, delegates, timers, tasks, subscriptions, and caches. Swift 6.x strict memory safety helps unsafe low-level memory access, but ARC ownership issues still need architectural discipline.

## Practice

1. Create a class with `deinit` and observe when it prints.
2. Create two strong references to the same object.
3. Explain why ARC is not garbage collection.
4. Explain why value types are not managed like class instances.
