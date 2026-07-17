# Day 16: Closures And Memory Leaks

## Why Closures Leak

Closures are reference types. When a closure captures a class instance, it captures strongly by default.

```swift
final class ProfileViewModel {
    var onReload: (() -> Void)?

    func setup() {
        onReload = {
            self.load()
        }
    }

    func load() { }
}
```

The view model owns `onReload`, and `onReload` owns `self`. This is a cycle.

## Closure Capture Rules

By default:

- Capturing a class instance is strong.
- Capturing a struct captures value semantics.
- Escaping closures can outlive the function call.
- Stored closures often outlive the immediate scope.

This is why escaping and stored closures need special attention.

## Fix With Capture List

```swift
onReload = { [weak self] in
    self?.load()
}
```

## Weak Self With Guard

```swift
service.fetch { [weak self] result in
    guard let self else { return }
    self.handle(result)
}
```

This pattern is common when the closure has multiple lines.

## Capture Values Instead Of Self

Sometimes you do not need `self`.

```swift
let userId = self.userId
service.fetchUser(id: userId) { result in
    print(result)
}
```

Capturing only the needed value reduces lifetime coupling.

## Capture Lists For Values

```swift
var count = 0

let printCount = { [count] in
    print(count)
}

count = 10
printCount() // prints 0
```

The capture list captured the value at closure creation time.

Senior point:

Capture lists are not only for `[weak self]`; they can intentionally freeze values.

## Timers

Timers can create cycles.

```swift
timer = Timer.scheduledTimer(withTimeInterval: 1, repeats: true) { [weak self] _ in
    self?.tick()
}
```

Invalidate timers when no longer needed.

## Tasks

Swift concurrency tasks can also affect object lifetime.

```swift
task = Task { [weak self] in
    await self?.load()
}
```

Cancel long-running tasks when appropriate.

## `@Sendable` And Modern Swift

Closures that cross concurrency boundaries may need to be `@Sendable`.

```swift
let operation: @Sendable () -> Void = {
    print("Safe to send across concurrency domains")
}
```

`@Sendable` is more about data-race safety than retain cycles, but both matter when closures run asynchronously.

## Advanced Closure Ownership Questions

Ask these during review:

1. Is the closure stored?
2. Is the closure escaping?
3. Does the closure capture `self`?
4. Does `self` own the closure?
5. Should the closure keep `self` alive?
6. Should the work cancel when `self` disappears?
7. Is the closure crossing actor or task boundaries?

## SwiftUI And UIKit Examples

UIKit callback:

```swift
UIView.animate(withDuration: 0.3) { [weak self] in
    self?.view.alpha = 0
}
```

SwiftUI action:

```swift
Button("Refresh") {
    Task {
        await viewModel.refresh()
    }
}
```

SwiftUI often uses value-type views, but reference-type view models can still leak through closures or tasks.

## Junior-Level Interview Answer

Closures can cause memory leaks because they capture objects strongly by default.

## Mid-Level Interview Answer

If an object stores a closure and that closure captures the object, a retain cycle can happen. Use `[weak self]` or `[unowned self]` depending on lifetime.

## Senior-Level Interview Answer

Closure memory management is about capture ownership. I ask whether the closure should keep `self` alive. For UI callbacks and optional lifetimes, weak is common. For operations that must complete and intentionally own their context, strong capture may be correct. I also consider tasks, timers, Combine subscriptions, and cancellation.

## Common Mistakes

- Using `[weak self]` without handling nil.
- Using `[unowned self]` in async work.
- Capturing all of `self` when only one value is needed.
- Forgetting to cancel tasks or invalidate timers.

## Interview Progression

Basic:

Closures can capture objects and keep them alive.

Intermediate:

Stored escaping closures that capture `self` can create retain cycles. Use capture lists to control ownership.

Advanced:

Closure lifetime must match business semantics. Weak capture is correct when work should stop if the owner disappears. Strong capture may be intentional when the operation owns its context. In concurrent Swift, also consider `@Sendable`, actor isolation, and cancellation.

## Practice

1. Create a closure property that leaks.
2. Fix it with `[weak self]`.
3. Capture a value instead of self.
4. Explain closure lifetime in a network callback.
