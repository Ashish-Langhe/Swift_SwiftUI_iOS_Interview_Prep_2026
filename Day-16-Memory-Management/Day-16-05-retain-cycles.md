# Day 16: Retain Cycles

## Core Idea

A retain cycle happens when objects keep each other alive through strong references.

```swift
final class Person {
    var apartment: Apartment?

    deinit {
        print("Person deallocated")
    }
}

final class Apartment {
    var tenant: Person?

    deinit {
        print("Apartment deallocated")
    }
}
```

If `Person` strongly owns `Apartment` and `Apartment` strongly owns `Person`, ARC cannot deallocate either.

## How To See The Cycle

```text
Person -> Apartment
Apartment -> Person
```

Both arrows are strong. ARC sees each object still has a strong owner, so neither reference count reaches zero.

## Fix With Weak

```swift
final class Apartment {
    weak var tenant: Person?
}
```

Now `Apartment` does not keep `Person` alive.

## Common Retain Cycle Patterns

### Parent And Child

Parent strongly owns child. Child should weakly reference parent.

```swift
final class ParentCoordinator {
    var child: ChildCoordinator?
}

final class ChildCoordinator {
    weak var parent: ParentCoordinator?
}
```

### Delegate

Owner should not strongly own delegate.

```swift
weak var delegate: PaymentDelegate?
```

### Closure

Object owns closure. Closure strongly captures object.

```swift
final class ViewModel {
    var onUpdate: (() -> Void)?

    func bind() {
        onUpdate = {
            self.refresh()
        }
    }
}
```

Fix:

```swift
onUpdate = { [weak self] in
    self?.refresh()
}
```

### Timer

Repeating timers can keep closures alive.

```swift
timer = Timer.scheduledTimer(withTimeInterval: 1, repeats: true) { [weak self] _ in
    self?.tick()
}
```

Invalidate timers when appropriate.

### Notification Observer

Block-based observers can retain captured objects.

```swift
observer = NotificationCenter.default.addObserver(
    forName: UIApplication.didBecomeActiveNotification,
    object: nil,
    queue: .main
) { [weak self] _ in
    self?.refresh()
}
```

If you store the observer token, remove it when it is no longer needed.

### Combine Subscription

```swift
cancellable = publisher
    .sink { [weak self] value in
        self?.handle(value)
    }
```

If `self` owns `cancellable`, and the sink closure owns `self`, you have a cycle.

### Task

```swift
task = Task { [weak self] in
    await self?.load()
}
```

Tasks can keep work alive. Cancel long-running tasks intentionally.

## Real iOS Use Cases

Retain cycles commonly appear in:

- Coordinators
- Delegates
- View models with callbacks
- Timers
- Notification observers
- Combine subscriptions
- Long-running tasks

## Debugging Flow

1. Add `deinit` logs to the suspected object.
2. Navigate to the screen.
3. Dismiss or pop the screen.
4. Check if `deinit` prints.
5. If not, open Memory Graph.
6. Search for the object.
7. Inspect the strong reference path.
8. Break the incorrect ownership edge.
9. Retest.

## Junior-Level Interview Answer

A retain cycle happens when two objects hold strong references to each other, so ARC cannot free them.

## Mid-Level Interview Answer

Retain cycles are fixed by breaking one strong reference using `weak` or `unowned`. Delegates are usually weak. Closures often use `[weak self]`.

## Senior-Level Interview Answer

Retain cycle debugging is ownership debugging. I identify who should own whom, then make back-references weak. I do not apply weak randomly; I preserve necessary lifetimes while breaking cycles. I confirm fixes with `deinit`, Xcode Memory Graph, and Instruments.

## Common Mistakes

- Strong delegates.
- Coordinator child strongly references parent.
- Timer strongly retains target.
- Closure properties capture `self`.
- Combine sink captures `self` without weak capture.

## Interview Progression

Basic:

A retain cycle is when objects keep each other alive.

Intermediate:

Break cycles using weak or unowned on the non-owning side.

Advanced:

Retain-cycle fixes require ownership analysis. The goal is not "add weak everywhere"; the goal is to make the ownership graph accurately represent lifecycle. Verify with `deinit`, Memory Graph, and Instruments.

## Practice

1. Create a two-object retain cycle.
2. Fix it with weak.
3. Create closure retain cycle.
4. Add deinit logs to verify release.
