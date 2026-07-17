# Day 26: Forms

## What `Form` Is

`Form` is SwiftUI's structured container for data entry and settings-style screens.

Use it for:

- Login
- Profile editing
- Settings
- Preferences
- Checkout details
- Admin tools
- Feedback forms

```swift
struct ProfileForm: View {
    @State private var name = ""
    @State private var isPublic = true

    var body: some View {
        Form {
            TextField("Name", text: $name)
            Toggle("Public profile", isOn: $isPublic)
        }
    }
}
```

`Form` automatically adapts row styling and layout to the platform.

## Form State Should Usually Be a Draft

Do not mutate saved domain data on every keystroke unless that is intentional.

Better:

```swift
struct ProfileDraft {
    var displayName: String
    var bio: String
    var isPublic: Bool

    init(profile: Profile) {
        displayName = profile.displayName
        bio = profile.bio
        isPublic = profile.isPublic
    }
}
```

Use it:

```swift
struct EditProfileScreen: View {
    let profile: Profile
    let save: (ProfileDraft) async throws -> Void

    @State private var draft: ProfileDraft
    @State private var isSaving = false
    @State private var errorMessage: String?

    init(profile: Profile, save: @escaping (ProfileDraft) async throws -> Void) {
        self.profile = profile
        self.save = save
        _draft = State(initialValue: ProfileDraft(profile: profile))
    }

    var body: some View {
        Form {
            Section("Profile") {
                TextField("Display name", text: $draft.displayName)
                TextField("Bio", text: $draft.bio, axis: .vertical)
                    .lineLimit(3...6)
                Toggle("Public profile", isOn: $draft.isPublic)
            }

            Section {
                Button("Save") {
                    Task { await submit() }
                }
                .disabled(!canSave || isSaving)
            }
        }
    }

    private var canSave: Bool {
        !draft.displayName.trimmingCharacters(in: .whitespacesAndNewlines).isEmpty
    }

    @MainActor
    private func submit() async {
        isSaving = true
        defer { isSaving = false }

        do {
            try await save(draft)
        } catch {
            errorMessage = "Could not save profile."
        }
    }
}
```

This protects the source model until the user commits.

## Sections and Footers

```swift
Form {
    Section {
        Toggle("Face ID", isOn: $faceIDEnabled)
    } footer: {
        Text("Use Face ID to unlock the app faster.")
    }
}
```

Footers should explain consequences, privacy, billing, or security behavior.

## Validation

Inline validation works well for forms.

```swift
struct EmailField: View {
    @Binding var email: String

    var isValid: Bool {
        email.contains("@") && email.contains(".")
    }

    var body: some View {
        Section {
            TextField("Email", text: $email)
                .keyboardType(.emailAddress)
                .textInputAutocapitalization(.never)
                .autocorrectionDisabled()

            if !email.isEmpty && !isValid {
                Text("Enter a valid email address.")
                    .font(.caption)
                    .foregroundStyle(.red)
            }
        }
    }
}
```

Senior tip: validation rules belong near the model or domain if they are business-critical.

## Focus Management

```swift
enum LoginField {
    case email
    case password
}

struct LoginForm: View {
    @State private var email = ""
    @State private var password = ""
    @FocusState private var focusedField: LoginField?

    var body: some View {
        Form {
            TextField("Email", text: $email)
                .focused($focusedField, equals: .email)
                .submitLabel(.next)

            SecureField("Password", text: $password)
                .focused($focusedField, equals: .password)
                .submitLabel(.go)
        }
        .onSubmit {
            switch focusedField {
            case .email:
                focusedField = .password
            case .password:
                signIn()
            case nil:
                break
            }
        }
    }
}
```

Good forms support keyboard flow.

## Pickers in Forms

```swift
enum NotificationFrequency: String, CaseIterable, Identifiable {
    case never
    case daily
    case weekly

    var id: String { rawValue }
}

Picker("Notifications", selection: $frequency) {
    ForEach(NotificationFrequency.allCases) { frequency in
        Text(frequency.rawValue.capitalized)
            .tag(frequency)
    }
}
```

Use domain enums for valid choices.

## Form Submission Strategy

Strong form flows handle:

- Dirty state
- Validation
- Loading
- Error presentation
- Cancel/discard confirmation
- Keyboard focus
- Accessibility labels
- Server-side validation failures

Example dirty check:

```swift
var hasChanges: Bool {
    draft != ProfileDraft(profile: profile)
}
```

## Common Mistakes

- Mutating persisted model state before save.
- No validation until backend failure.
- Too many booleans for form states.
- Buttons enabled during duplicate submission.
- No keyboard/focus flow.
- No unsaved-changes handling.
- Business validation hidden inside a view.

## Senior iOS Engineer Perspective

A senior engineer treats a form as a state machine:

- Initial state
- Editing state
- Invalid state
- Saving state
- Failed save state
- Saved state
- Discard confirmation

The UI is only one part. The correctness is in transitions.

## Interview Notes

Junior:

`Form` organizes inputs like text fields, toggles, pickers, and buttons.

Mid-level:

Use draft state, validation, sections, focus management, and disabled save buttons.

Senior:

I design forms around data ownership, validation boundaries, async submission, dirty-state handling, accessibility, and clear user recovery from errors.

## Practice

1. Build a profile edit form with a draft model.
2. Add inline email validation.
3. Add `FocusState` for keyboard flow.
4. Add unsaved-changes detection.
