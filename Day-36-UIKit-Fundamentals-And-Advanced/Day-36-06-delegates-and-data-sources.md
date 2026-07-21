# Day 36.6: Delegates And Data Sources

Delegates and data sources are foundational UIKit patterns. They predate SwiftUI, Combine, and async/await, but they remain everywhere in UIKit: table views, collection views, text fields, scroll views, navigation controllers, gesture recognizers, and custom components.

## Core Idea

A data source answers: "What data should I display?"

A delegate answers: "What should happen when something occurs?"

Example:

```swift
tableView.dataSource = self
tableView.delegate = self
```

## Data Source Responsibilities

For a table view:

```swift
extension OrdersViewController: UITableViewDataSource {
    func numberOfSections(in tableView: UITableView) -> Int {
        sections.count
    }

    func tableView(_ tableView: UITableView, numberOfRowsInSection section: Int) -> Int {
        sections[section].rows.count
    }

    func tableView(_ tableView: UITableView, cellForRowAt indexPath: IndexPath) -> UITableViewCell {
        let row = sections[indexPath.section].rows[indexPath.row]
        let cell = tableView.dequeueReusableCell(withIdentifier: OrderCell.reuseID, for: indexPath) as! OrderCell
        cell.configure(with: row)
        return cell
    }
}
```

The data source should be deterministic. Given the current state and index path, it should return the correct cell.

## Delegate Responsibilities

```swift
extension OrdersViewController: UITableViewDelegate {
    func tableView(_ tableView: UITableView, didSelectRowAt indexPath: IndexPath) {
        tableView.deselectRow(at: indexPath, animated: true)
        let order = sections[indexPath.section].rows[indexPath.row]
        coordinator.showOrderDetails(order.id)
    }
}
```

Delegates commonly handle:

- Selection.
- Editing actions.
- Scroll events.
- Layout decisions.
- Navigation decisions.
- Validation events.

## Weak Delegates

Delegates are usually weak to avoid retain cycles.

```swift
protocol CheckoutSummaryViewDelegate: AnyObject {
    func checkoutSummaryViewDidTapPay(_ view: CheckoutSummaryView)
}

final class CheckoutSummaryView: UIView {
    weak var delegate: CheckoutSummaryViewDelegate?

    @objc private func payTapped() {
        delegate?.checkoutSummaryViewDidTapPay(self)
    }
}
```

`AnyObject` restricts the protocol to class types so `weak` is allowed.

## Custom Delegate vs Closure

Delegate:

```swift
protocol FilterViewDelegate: AnyObject {
    func filterView(_ view: FilterView, didSelect filter: Filter)
}
```

Closure:

```swift
final class FilterView: UIView {
    var onSelectFilter: ((Filter) -> Void)?
}
```

Use delegate when:

- Multiple events are related.
- The component has a formal lifecycle.
- You want Objective-C/UIKit style consistency.
- You need weak ownership by default.

Use closure when:

- There is one simple event.
- The view is small.
- You want lightweight composition.

Senior caution: closures capture strongly by default.

```swift
filterView.onSelectFilter = { [weak self] filter in
    self?.apply(filter)
}
```

## Delegate Method Naming

Good names include sender and event:

```swift
func paymentMethodView(_ view: PaymentMethodView, didSelect method: PaymentMethod)
func paymentMethodViewDidTapAddCard(_ view: PaymentMethodView)
```

Avoid vague names:

```swift
func selected(_ value: String)
func didTap()
```

Clear delegate APIs are part of framework-quality UIKit.

## Data Source As Separate Object

For complex screens, the view controller does not need to be the data source.

```swift
final class OrdersDataSource: NSObject, UITableViewDataSource {
    private var sections: [OrdersSection] = []

    func update(sections: [OrdersSection]) {
        self.sections = sections
    }

    func tableView(_ tableView: UITableView, numberOfRowsInSection section: Int) -> Int {
        sections[section].rows.count
    }

    func tableView(_ tableView: UITableView, cellForRowAt indexPath: IndexPath) -> UITableViewCell {
        // Configure cell
        UITableViewCell()
    }
}
```

This helps when:

- The controller is too large.
- Multiple controllers share list rendering.
- You need isolated testing.

Diffable data source is often a better modern version of this idea.

## Scroll View Delegate

`UIScrollViewDelegate` powers many interactions.

```swift
func scrollViewDidScroll(_ scrollView: UIScrollView) {
    let offset = scrollView.contentOffset.y
    headerView.alpha = max(0, 1 - offset / 120)
}
```

Senior caution:

- `scrollViewDidScroll` can run many times per second.
- Avoid heavy work.
- Debounce analytics.
- Do not trigger layout storms.

## Text Field Delegate

```swift
extension LoginViewController: UITextFieldDelegate {
    func textFieldShouldReturn(_ textField: UITextField) -> Bool {
        if textField === emailField {
            passwordField.becomeFirstResponder()
        } else {
            signIn()
        }
        return true
    }
}
```

Use for keyboard flow, validation, and editing behavior.

## Delegate Ownership And Retain Cycles

Bad:

```swift
class ChildView: UIView {
    var delegate: ChildViewDelegate?
}
```

If the parent owns the child and the child strongly owns the parent as delegate, both leak.

Better:

```swift
weak var delegate: ChildViewDelegate?
```

## Common Mistakes

- Making delegate strong.
- Putting data source mutation inside cell configuration.
- Calling delegate methods without useful context.
- Creating too many one-method protocols when a closure is enough.
- Using closures without capture lists.
- Doing expensive work in scroll delegate callbacks.

## Junior-Level Interview Answer

A data source provides data and cells. A delegate handles events and behavior. In UIKit, many controls use this pattern. Delegates are usually weak to avoid retain cycles.

## Senior-Level Interview Answer

Delegation is a boundary pattern. I use it to invert control from reusable views back to coordinators or controllers without making the child know about navigation or business logic. I keep data sources deterministic, delegates weak, callbacks semantically named, and choose closures for small single-event components.

## Points To Remember

- Data source means content.
- Delegate means behavior.
- Delegates should usually be `weak`.
- Delegate protocols for views should inherit `AnyObject`.
- Scroll delegates are performance-sensitive.
- Modern diffable data source reduces manual data-source complexity.

