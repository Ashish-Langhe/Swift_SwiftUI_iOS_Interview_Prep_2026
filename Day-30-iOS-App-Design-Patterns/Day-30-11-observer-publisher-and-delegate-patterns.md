# Day 30: Observer, Publisher, And Delegate Patterns

## Why Communication Patterns Matter

iOS apps need objects to communicate:

- ViewController to delegate
- Model to observers
- Service to subscribers
- Child to parent
- UIKit control to SwiftUI binding

Choosing the right communication pattern prevents tight coupling.

## Delegate Pattern

Delegate is one-to-one communication.

```swift
protocol ImagePickerDelegate: AnyObject {
    func imagePickerDidSelectImage(_ image: UIImage)
    func imagePickerDidCancel()
}

final class ImagePickerViewController: UIViewController {
    weak var delegate: ImagePickerDelegate?
}
```

Use weak delegate to avoid retain cycles.

Good for:

- UIKit components
- Child-to-parent events
- Custom controls
- Coordinators

## Observer Pattern

Observer is one-to-many notification.

```swift
NotificationCenter.default.post(name: .sessionExpired, object: nil)
```

Use carefully. NotificationCenter can become invisible global messaging.

## Combine Publisher

```swift
final class SessionStore {
    @Published private(set) var session: UserSession?
}
```

Subscriber:

```swift
sessionStore.$session
    .sink { session in
        // react
    }
    .store(in: &cancellables)
```

Use Combine when stream composition matters or legacy code already uses it.

## AsyncSequence

Modern concurrency can model event streams.

```swift
protocol MessagesClient {
    var messages: AsyncStream<Message> { get }
}
```

Consume:

```swift
for await message in messagesClient.messages {
    messages.append(message)
}
```

## SwiftUI Observation

```swift
@Observable
final class SessionModel {
    var user: User?
}
```

SwiftUI observes property access and updates views.

Good for UI state. Not a replacement for every event stream.

## Delegate vs Closure

Closure is simpler for one event.

```swift
struct AddTaskView: View {
    let onSave: (TaskDraft) -> Void
}
```

Delegate is useful for multiple related events.

```swift
protocol AddTaskDelegate: AnyObject {
    func addTaskDidSave(_ draft: TaskDraft)
    func addTaskDidCancel()
}
```

## When To Use What

Delegate:

- One-to-one, UIKit-style, multiple callbacks.

Closure:

- One or two simple callbacks.

Notification:

- System-wide event with loose coupling, use sparingly.

Combine:

- Reactive streams, debouncing, composition.

AsyncSequence:

- Async event streams with concurrency.

Observation:

- SwiftUI observable UI state.

## Common Mistakes

- Strong delegates causing retain cycles.
- NotificationCenter for everything.
- Combine pipelines with unclear ownership.
- Closures capturing `self` strongly.
- Using Observation for one-shot events.
- Multiple communication patterns for same flow.

## Senior Artifact

```text
Artifact: Communication Pattern Review
Event:
Sender:
Receiver(s):
One-to-one or one-to-many:
Synchronous or async:
Lifetime:
Memory risk:
Chosen pattern:
```

## Interview Notes

Junior:

Delegate lets one object notify another.

Mid-level:

Use delegates/closures for direct communication, publishers or async streams for ongoing events.

Senior:

I choose communication patterns based on cardinality, lifetime, ownership, memory risk, and testability. I avoid global notification soup and retain cycles.

## Practice

1. Build a delegate-based image picker.
2. Replace a simple delegate with a closure.
3. Model a message stream with `AsyncStream`.
4. Identify a closure retain cycle.
