# Day 36.12: UIView, CALayer, Drawing, And Rendering

UIKit views are backed by Core Animation layers. A senior iOS engineer should understand the difference between `UIView` and `CALayer`, when custom drawing is appropriate, why shadows and masks can affect performance, and how rendering decisions show up in scrolling performance.

## UIView vs CALayer

`UIView`:

- Handles interaction.
- Participates in Auto Layout.
- Owns accessibility.
- Manages a backing layer.
- Has UIKit lifecycle and responder behavior.

`CALayer`:

- Handles visual backing.
- Supports animations.
- Has properties like `cornerRadius`, `shadowPath`, `borderWidth`, `contents`.
- Does not handle UIKit events directly.

Example:

```swift
avatarImageView.layer.cornerRadius = 32
avatarImageView.layer.masksToBounds = true
```

## Common Layer Styling

```swift
cardView.layer.cornerRadius = 12
cardView.layer.borderWidth = 1
cardView.layer.borderColor = UIColor.separator.cgColor
```

Shadow:

```swift
cardView.layer.shadowColor = UIColor.black.cgColor
cardView.layer.shadowOpacity = 0.12
cardView.layer.shadowRadius = 12
cardView.layer.shadowOffset = CGSize(width: 0, height: 4)
cardView.layer.shadowPath = UIBezierPath(
    roundedRect: cardView.bounds,
    cornerRadius: 12
).cgPath
```

Set `shadowPath` when possible. It helps Core Animation avoid calculating the shadow from dynamic alpha content every frame.

## Corner Radius Plus Shadow

A common issue: `masksToBounds = true` clips shadows.

Bad:

```swift
cardView.layer.cornerRadius = 12
cardView.layer.masksToBounds = true
cardView.layer.shadowOpacity = 0.2
```

Better structure:

```swift
let shadowContainer = UIView()
let roundedContent = UIView()

shadowContainer.layer.shadowOpacity = 0.15
shadowContainer.layer.shadowRadius = 10
shadowContainer.layer.shadowOffset = CGSize(width: 0, height: 4)

roundedContent.layer.cornerRadius = 12
roundedContent.layer.masksToBounds = true
```

The outer view draws shadow. The inner view clips rounded content.

## Custom Drawing With `draw(_:)`

Use custom drawing for lightweight vector-like rendering that depends on bounds.

```swift
final class ProgressRingView: UIView {
    var progress: CGFloat = 0 {
        didSet { setNeedsDisplay() }
    }

    override func draw(_ rect: CGRect) {
        let lineWidth: CGFloat = 8
        let radius = min(rect.width, rect.height) / 2 - lineWidth
        let center = CGPoint(x: rect.midX, y: rect.midY)

        let path = UIBezierPath(
            arcCenter: center,
            radius: radius,
            startAngle: -.pi / 2,
            endAngle: -.pi / 2 + progress * 2 * .pi,
            clockwise: true
        )

        UIColor.systemBlue.setStroke()
        path.lineWidth = lineWidth
        path.lineCapStyle = .round
        path.stroke()
    }
}
```

Senior caution:

- `draw(_:)` runs on the main thread.
- Do not decode images or do expensive calculations inside `draw`.
- Call `setNeedsDisplay()` to invalidate drawing.
- Prefer layers or `CAShapeLayer` for animatable shapes.

## CAShapeLayer

Often better than drawing in `draw(_:)` for shapes:

```swift
let shapeLayer = CAShapeLayer()
shapeLayer.strokeColor = UIColor.systemBlue.cgColor
shapeLayer.fillColor = UIColor.clear.cgColor
shapeLayer.lineWidth = 8
shapeLayer.lineCap = .round
shapeLayer.path = UIBezierPath(ovalIn: bounds.insetBy(dx: 8, dy: 8)).cgPath
layer.addSublayer(shapeLayer)
```

Use `CAShapeLayer` for:

- Rings.
- Paths.
- Borders.
- Badges.
- Lightweight vector shapes.

## Rasterization

Layer rasterization caches a layer into a bitmap.

```swift
cardView.layer.shouldRasterize = true
cardView.layer.rasterizationScale = UIScreen.main.scale
```

Use carefully. It can help when a complex layer is static, but hurt when the layer changes frequently.

Senior rule: never turn on rasterization blindly. Measure.

## Offscreen Rendering

Offscreen rendering may happen when the system needs an intermediate buffer for effects like:

- Shadows without shadow path.
- Masks.
- Group opacity.
- Some rounded corner combinations.
- Visual effect views.

Not all offscreen rendering is bad. It is a problem when it causes dropped frames, especially in scrolling lists.

## Images And Rendering

Use correct image sizes. Do not load a huge image into a tiny cell.

```swift
func downsample(image: UIImage, targetSize: CGSize) -> UIImage {
    // In production, use ImageIO downsampling for memory efficiency.
    UIGraphicsImageRenderer(size: targetSize).image { _ in
        image.draw(in: CGRect(origin: .zero, size: targetSize))
    }
}
```

Senior thinking:

- Decode/downsample before displaying.
- Cache appropriately.
- Avoid huge bitmap memory spikes.
- Assign images on main actor after background processing.

## Animations And Layers

UIKit property animation:

```swift
UIView.animate(withDuration: 0.25) {
    self.cardView.alpha = 0
    self.cardView.transform = CGAffineTransform(scaleX: 0.95, y: 0.95)
}
```

Layer animation:

```swift
let animation = CABasicAnimation(keyPath: "transform.scale")
animation.fromValue = 0.95
animation.toValue = 1.0
animation.duration = 0.2
cardView.layer.add(animation, forKey: "scale")
```

UIKit animations update model values. Core Animation can animate presentation-layer values. Know the difference when debugging final state.

## Common Mistakes

- Combining shadow and `masksToBounds` on the same layer.
- Drawing expensive content in `draw(_:)`.
- Forgetting to update `shadowPath` after bounds change.
- Using huge images in small views.
- Animating layout without calling `layoutIfNeeded`.
- Enabling rasterization without measurement.

## Junior-Level Interview Answer

`UIView` manages layout, events, and accessibility. It has a backing `CALayer` that handles visual properties like corner radius, border, and shadow. Custom drawing can be done in `draw(_:)`, but should be used carefully.

## Senior-Level Interview Answer

I think of rendering as a performance budget. Views handle UIKit behavior, layers handle compositing and animation, and custom drawing should be minimal and invalidated intentionally. For shadows, I set `shadowPath`; for rounded content plus shadows, I split containers; for cells, I avoid expensive drawing, huge images, and unnecessary offscreen rendering unless profiling shows it is acceptable.

## Points To Remember

- `UIView` is interaction plus layout plus accessibility.
- `CALayer` is rendering and animation backing.
- `masksToBounds` clips shadows.
- `shadowPath` improves shadow performance.
- `draw(_:)` is powerful but easy to misuse.
- Rendering problems often appear as scroll jank.

