# Day 36.16: UIKit Animations And Transitions

UIKit animation knowledge is essential for polished interfaces and advanced interviews. A junior engineer should know `UIView.animate`. A senior engineer should understand layout animations, property animators, interruptibility, transition coordination, list updates, reduced motion, and performance.

## Basic UIView Animation

```swift
UIView.animate(withDuration: 0.25) {
    self.bannerView.alpha = 1
    self.bannerView.transform = .identity
}
```

Common animatable properties:

- `alpha`
- `transform`
- `center`
- `frame`
- `bounds`
- `backgroundColor`
- constraint-driven layout changes

## Constraint Animation

Update constraint constants, then animate `layoutIfNeeded`.

```swift
bottomConstraint.constant = isVisible ? -16 : 120

UIView.animate(withDuration: 0.3) {
    self.view.layoutIfNeeded()
}
```

Common mistake: calling `layoutIfNeeded` before changing constraints or on the wrong parent view.

## Spring Animation

```swift
UIView.animate(
    withDuration: 0.6,
    delay: 0,
    usingSpringWithDamping: 0.75,
    initialSpringVelocity: 0.4,
    options: [.allowUserInteraction]
) {
    self.cardView.transform = .identity
}
```

Use spring animations for:

- Cards.
- Buttons.
- Drag release.
- Sheet-like UI.

Avoid spring animation where precision matters more than feel.

## UIViewPropertyAnimator

`UIViewPropertyAnimator` supports interactive and interruptible animations.

```swift
let animator = UIViewPropertyAnimator(duration: 0.35, dampingRatio: 0.8) {
    self.panelView.transform = CGAffineTransform(translationX: 0, y: -300)
}

animator.startAnimation()
```

Interactive:

```swift
animator.fractionComplete = progress
```

Use for:

- Draggable cards.
- Interactive panels.
- Scrubbable transitions.
- Interruptible UI changes.

## Transition Coordinator

Synchronize with navigation or modal transitions:

```swift
transitionCoordinator?.animate(alongsideTransition: { _ in
    self.headerView.alpha = 0
}, completion: { context in
    if context.isCancelled {
        self.headerView.alpha = 1
    }
})
```

Senior note: always consider cancellation for interactive gestures.

## Table And Collection Updates

Diffable snapshot:

```swift
dataSource.apply(snapshot, animatingDifferences: true)
```

Manual table update:

```swift
tableView.performBatchUpdates {
    rows.remove(at: index)
    tableView.deleteRows(at: [IndexPath(row: index, section: 0)], with: .automatic)
}
```

Senior preference: use diffable for state-driven list changes unless there is a strong reason for manual updates.

## Animating Cell Selection

```swift
override var isHighlighted: Bool {
    didSet {
        UIView.animate(withDuration: 0.15) {
            self.contentView.transform = self.isHighlighted
                ? CGAffineTransform(scaleX: 0.98, y: 0.98)
                : .identity
        }
    }
}
```

Keep cell animations lightweight.

## Reduced Motion

Respect accessibility settings.

```swift
if UIAccessibility.isReduceMotionEnabled {
    view.alpha = 1
} else {
    UIView.animate(withDuration: 0.25) {
        self.view.alpha = 1
    }
}
```

Senior thinking: animation should support comprehension, not block usability.

## Animation Performance

Prefer animating:

- `transform`
- `opacity`

Be careful animating:

- Constraints across complex hierarchies.
- Shadows.
- Masks.
- Corner radius in many cells.
- Blur effects.

Use Instruments/Core Animation when animation jank appears.

## Common Mistakes

- Animating constraints without `layoutIfNeeded`.
- Forgetting interactive transition cancellation.
- Too many simultaneous animations in cells.
- Animations that ignore reduced motion.
- Updating model state only in animation completion.
- Starting animations from inconsistent initial state.

## Junior-Level Interview Answer

UIKit animations can be created with `UIView.animate`. For Auto Layout, change constraint constants and animate `layoutIfNeeded`. Collection and table changes can be animated with diffable snapshots or batch updates.

## Senior-Level Interview Answer

I choose animation APIs based on interaction. Simple state changes use `UIView.animate`; interactive gestures use `UIViewPropertyAnimator`; navigation-related changes use transition coordinators; list changes use diffable snapshots. I handle cancellation, reduced motion, and performance by preferring transform/opacity and measuring complex effects.

## Points To Remember

- Change constraints, then animate layout.
- Property animators are interruptible.
- Transition coordinator handles navigation animation timing.
- Reduced Motion matters.
- Transform and opacity are usually cheaper than layout-heavy animations.

