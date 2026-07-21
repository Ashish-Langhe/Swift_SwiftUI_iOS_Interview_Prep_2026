# Day 36.15: UIKit Forms, Input, Keyboard, And Validation

Forms are deceptively hard. A basic form is easy to build; a production form must handle keyboard behavior, validation, errors, accessibility, autofill, secure text entry, localization, and network submission state.

## Form Mental Model

A form screen usually has:

- Input fields.
- Validation rules.
- Error presentation.
- Keyboard navigation.
- Submit button state.
- Loading state.
- Accessibility support.
- Secure input rules where needed.

## Basic Form

```swift
final class LoginViewController: UIViewController {
    private let emailField = UITextField()
    private let passwordField = UITextField()
    private let submitButton = UIButton(type: .system)

    override func viewDidLoad() {
        super.viewDidLoad()
        configureFields()
        configureSubmitButton()
    }

    private func configureFields() {
        emailField.placeholder = "Email"
        emailField.keyboardType = .emailAddress
        emailField.textContentType = .username
        emailField.autocapitalizationType = .none
        emailField.autocorrectionType = .no
        emailField.returnKeyType = .next
        emailField.delegate = self

        passwordField.placeholder = "Password"
        passwordField.textContentType = .password
        passwordField.isSecureTextEntry = true
        passwordField.returnKeyType = .go
        passwordField.delegate = self
    }
}
```

## Text Content Types

Set `textContentType` for autofill and system intelligence:

```swift
emailField.textContentType = .emailAddress
passwordField.textContentType = .password
oneTimeCodeField.textContentType = .oneTimeCode
newPasswordField.textContentType = .newPassword
```

This improves:

- Password manager support.
- SMS code autofill.
- Keyboard suggestions.
- User experience.

## Keyboard Avoidance

Modern UIKit has `keyboardLayoutGuide`.

```swift
NSLayoutConstraint.activate([
    formStack.bottomAnchor.constraint(
        lessThanOrEqualTo: view.keyboardLayoutGuide.topAnchor,
        constant: -20
    )
])
```

This is often cleaner than manual keyboard notification handling.

## Scrollable Forms

For longer forms:

```swift
let scrollView = UIScrollView()
let contentView = UIView()

NSLayoutConstraint.activate([
    contentView.topAnchor.constraint(equalTo: scrollView.contentLayoutGuide.topAnchor),
    contentView.leadingAnchor.constraint(equalTo: scrollView.contentLayoutGuide.leadingAnchor),
    contentView.trailingAnchor.constraint(equalTo: scrollView.contentLayoutGuide.trailingAnchor),
    contentView.bottomAnchor.constraint(equalTo: scrollView.contentLayoutGuide.bottomAnchor),
    contentView.widthAnchor.constraint(equalTo: scrollView.frameLayoutGuide.widthAnchor)
])
```

Use this with keyboard layout guide to keep fields visible.

## Return Key Flow

```swift
extension LoginViewController: UITextFieldDelegate {
    func textFieldShouldReturn(_ textField: UITextField) -> Bool {
        if textField === emailField {
            passwordField.becomeFirstResponder()
        } else {
            passwordField.resignFirstResponder()
            submit()
        }
        return true
    }
}
```

Small form details matter in interviews because they reveal product sensibility.

## Validation

Do not mix validation rules directly into UI controls.

```swift
struct LoginValidator {
    func validate(email: String, password: String) -> LoginValidationResult {
        guard email.contains("@") else {
            return .invalidEmail
        }

        guard password.count >= 8 else {
            return .passwordTooShort
        }

        return .valid
    }
}

enum LoginValidationResult {
    case valid
    case invalidEmail
    case passwordTooShort
}
```

Render errors:

```swift
func render(_ result: LoginValidationResult) {
    switch result {
    case .valid:
        errorLabel.text = nil
        submitButton.isEnabled = true
    case .invalidEmail:
        errorLabel.text = "Enter a valid email address."
        submitButton.isEnabled = false
    case .passwordTooShort:
        errorLabel.text = "Password must be at least 8 characters."
        submitButton.isEnabled = false
    }
}
```

## Real-Time vs Submit Validation

Real-time validation:

- Good for format feedback.
- Useful for password strength.
- Can feel noisy if too aggressive.

Submit validation:

- Better for forms where errors are expensive to show constantly.
- Useful when server validation is required.

Senior rule: validate at the right time for the user's mental model.

## Input Accessory View

```swift
let toolbar = UIToolbar()
toolbar.items = [
    UIBarButtonItem(systemItem: .flexibleSpace),
    UIBarButtonItem(title: "Done", style: .done, target: self, action: #selector(doneTapped))
]
toolbar.sizeToFit()
amountField.inputAccessoryView = toolbar
```

Use for numeric keyboards that do not have a Return key.

## Secure Input

```swift
passwordField.isSecureTextEntry = true
passwordField.textContentType = .password
```

For sensitive fields:

- Avoid logging entered values.
- Avoid storing raw values unnecessarily.
- Clear temporary values after submit if appropriate.
- Consider screenshots/app switcher privacy for highly sensitive screens.

## Submit State

```swift
func setSubmitting(_ isSubmitting: Bool) {
    submitButton.isEnabled = !isSubmitting
    emailField.isEnabled = !isSubmitting
    passwordField.isEnabled = !isSubmitting

    if isSubmitting {
        activityIndicator.startAnimating()
    } else {
        activityIndicator.stopAnimating()
    }
}
```

Prevent duplicate submissions.

## Accessibility For Forms

```swift
emailField.accessibilityLabel = "Email address"
passwordField.accessibilityLabel = "Password"
errorLabel.accessibilityTraits = [.staticText]
```

When an error appears:

```swift
UIAccessibility.post(notification: .announcement, argument: errorLabel.text)
```

## Common Mistakes

- No keyboard avoidance.
- Numeric keyboard without Done button.
- Duplicate submit requests.
- Validation only on client side for security-critical rules.
- Not setting text content type.
- Error messages not accessible.
- Password field not secure.
- Form state spread across many booleans.

## Junior-Level Interview Answer

UIKit forms use text fields, delegates, keyboard types, return key handling, and validation. I configure fields for the right keyboard, move focus between fields, show errors, and enable submit only when input is valid.

## Senior-Level Interview Answer

I treat forms as state machines. I separate validation from views, support autofill with content types, use keyboard layout guide or scroll views for keyboard avoidance, prevent duplicate submissions, make errors accessible, and distinguish client validation from server validation. For sensitive data, I avoid logs and unnecessary storage.

## Points To Remember

- Set keyboard type and text content type.
- Use return key flow.
- Use keyboard layout guide for avoidance.
- Keep validation testable.
- Prevent duplicate submit.
- Make errors accessible.

