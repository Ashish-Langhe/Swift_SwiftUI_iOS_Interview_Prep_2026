# Day 26: Alerts

## What Alerts Are

Alerts interrupt the user to communicate important information or confirm a decision.

Use alerts for:

- Destructive confirmation
- Error recovery
- Permission explanation
- Unsaved changes
- Critical decisions

Avoid alerts for routine information that can be shown inline.

## Basic Alert

```swift
struct DeleteAccountView: View {
    @State private var isShowingDeleteAlert = false

    var body: some View {
        Button("Delete Account", role: .destructive) {
            isShowingDeleteAlert = true
        }
        .alert("Delete Account?", isPresented: $isShowingDeleteAlert) {
            Button("Cancel", role: .cancel) {}
            Button("Delete", role: .destructive) {
                deleteAccount()
            }
        } message: {
            Text("This action cannot be undone.")
        }
    }
}
```

Keep alert copy direct and specific.

## Alert State as Data

For multiple alerts, use enum state.

```swift
enum ActiveAlert: Identifiable {
    case deleteProduct(Product.ID)
    case discardChanges
    case networkError(String)

    var id: String {
        switch self {
        case .deleteProduct(let id):
            return "delete-\(id)"
        case .discardChanges:
            return "discard"
        case .networkError(let message):
            return "network-\(message)"
        }
    }
}
```

Use:

```swift
.alert(item: $activeAlert) { alert in
    switch alert {
    case .deleteProduct(let id):
        return Alert(
            title: Text("Delete Product?"),
            message: Text("This product will be removed from the catalog."),
            primaryButton: .destructive(Text("Delete")) {
                deleteProduct(id)
            },
            secondaryButton: .cancel()
        )
    case .discardChanges:
        return Alert(
            title: Text("Discard Changes?"),
            primaryButton: .destructive(Text("Discard")) {
                discard()
            },
            secondaryButton: .cancel()
        )
    case .networkError(let message):
        return Alert(
            title: Text("Something Went Wrong"),
            message: Text(message),
            dismissButton: .default(Text("OK"))
        )
    }
}
```

Newer SwiftUI APIs provide more item/error-based alert overloads, making this state-driven style easier.

## Error Alerts

Model displayable errors.

```swift
struct DisplayError: Identifiable {
    let id = UUID()
    let title: String
    let message: String
}

@State private var displayError: DisplayError?

.alert(displayError?.title ?? "Error", item: $displayError) { _ in
    Button("OK", role: .cancel) {}
} message: { error in
    Text(error.message)
}
```

Avoid showing raw technical error messages to users.

## Confirmation Dialogs

Use confirmation dialogs for action choices.

```swift
.confirmationDialog("Photo Source", isPresented: $isShowingPhotoOptions) {
    Button("Camera") { openCamera() }
    Button("Photo Library") { openLibrary() }
    Button("Cancel", role: .cancel) {}
}
```

Use alerts for focused confirmation. Use confirmation dialogs for choosing among actions.

## Alerts and Async Work

```swift
Button("Retry") {
    Task {
        await model.reload()
    }
}
```

Do not start async work from hidden side effects. Make the user's action explicit.

## Destructive Alerts

A destructive alert should answer:

- What is being deleted?
- Is it reversible?
- What happens next?
- What is the safe cancel path?

```swift
.alert("Delete \(product.name)?", isPresented: $isConfirmingDelete) {
    Button("Cancel", role: .cancel) {}
    Button("Delete", role: .destructive) {
        delete(product)
    }
} message: {
    Text("This removes the product from all active listings.")
}
```

## Latest SwiftUI Notes

Recent SwiftUI updates add item/error-based alert and confirmation-dialog APIs. This aligns with a senior pattern: model alert presentation as data, then render the correct alert from that state.

## Common Mistakes

- Using alerts for non-critical messages.
- Keeping multiple boolean alert flags.
- Showing raw backend errors.
- No cancel option for destructive actions.
- Vague copy like "Are you sure?"
- Triggering alerts from inconsistent state.

## Senior iOS Engineer Perspective

Alerts are part of UX and state architecture:

- Model alert state explicitly.
- Use alerts sparingly.
- Prefer inline validation when possible.
- Make destructive copy specific.
- Use display-safe errors.
- Keep async retry paths testable.
- Ensure VoiceOver users receive meaningful titles and messages.

## Interview Notes

Junior:

Use `.alert` to show important messages or confirmations.

Mid-level:

Use enum or item-based alert state for multiple alerts.

Senior:

I treat alerts as explicit UI state, avoid alert spam, use domain-specific copy, protect destructive actions, and map technical errors into user-recoverable messages.

## Practice

1. Add a delete confirmation alert.
2. Replace two boolean alerts with an `ActiveAlert` enum.
3. Add a display-safe error alert.
4. Decide when a confirmation dialog is better than an alert.
