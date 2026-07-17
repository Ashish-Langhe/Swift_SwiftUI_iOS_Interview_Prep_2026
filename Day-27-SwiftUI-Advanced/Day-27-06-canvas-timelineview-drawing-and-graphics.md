# Day 27: Canvas, TimelineView, Drawing, And Graphics

## Why Drawing APIs Matter

Advanced SwiftUI sometimes needs custom rendering:

- Charts-like visuals
- Audio waveforms
- Timelines
- Custom gauges
- Animated backgrounds
- Particle effects
- Data visualization

SwiftUI gives higher-level drawing tools before you need Metal.

## `Shape`

Custom shape:

```swift
struct TicketShape: Shape {
    func path(in rect: CGRect) -> Path {
        var path = Path()
        let notchRadius: CGFloat = 12

        path.addRoundedRect(in: rect, cornerSize: CGSize(width: 12, height: 12))
        path.addEllipse(in: CGRect(
            x: rect.minX - notchRadius,
            y: rect.midY - notchRadius,
            width: notchRadius * 2,
            height: notchRadius * 2
        ))
        path.addEllipse(in: CGRect(
            x: rect.maxX - notchRadius,
            y: rect.midY - notchRadius,
            width: notchRadius * 2,
            height: notchRadius * 2
        ))

        return path
    }
}
```

## `Canvas`

`Canvas` gives immediate-mode drawing.

```swift
struct WaveformView: View {
    let samples: [CGFloat]

    var body: some View {
        Canvas { context, size in
            let midY = size.height / 2
            let step = size.width / CGFloat(max(samples.count - 1, 1))

            var path = Path()
            path.move(to: CGPoint(x: 0, y: midY))

            for index in samples.indices {
                let x = CGFloat(index) * step
                let y = midY - samples[index] * midY
                path.addLine(to: CGPoint(x: x, y: y))
            }

            context.stroke(path, with: .color(.blue), lineWidth: 2)
        }
        .frame(height: 120)
    }
}
```

Use `Canvas` for custom drawing where many SwiftUI subviews would be inefficient.

## `TimelineView`

Use `TimelineView` for time-driven updates.

```swift
struct ClockHandView: View {
    var body: some View {
        TimelineView(.periodic(from: .now, by: 1)) { timeline in
            let seconds = Calendar.current.component(.second, from: timeline.date)

            Rectangle()
                .frame(width: 2, height: 80)
                .rotationEffect(.degrees(Double(seconds) * 6))
        }
    }
}
```

Use it instead of manually creating timers in views.

## Canvas With Timeline

```swift
struct PulsingRings: View {
    var body: some View {
        TimelineView(.animation) { timeline in
            Canvas { context, size in
                let time = timeline.date.timeIntervalSinceReferenceDate
                let progress = time.truncatingRemainder(dividingBy: 1)
                let radius = 20 + progress * 80
                let opacity = 1 - progress

                let rect = CGRect(
                    x: size.width / 2 - radius,
                    y: size.height / 2 - radius,
                    width: radius * 2,
                    height: radius * 2
                )

                context.opacity = opacity
                context.stroke(Path(ellipseIn: rect), with: .color(.blue), lineWidth: 3)
            }
        }
        .frame(height: 220)
    }
}
```

## Mesh Gradients And Text Rendering

Recent SwiftUI releases continue expanding drawing and graphics APIs, including mesh gradients, shader compilation, custom text rendering effects, and color mixing. These are advanced visual tools, but they still need performance and accessibility review.

## Performance Notes

- Prefer `Canvas` for many simple drawing operations.
- Avoid animating too many independent views.
- Use `TimelineView` intentionally.
- Stop drawing when not visible if possible.
- Profile animation-heavy screens.
- Respect reduce motion.

## Common Mistakes

- Using hundreds of views for drawing that belongs in `Canvas`.
- Running manual timers in view bodies.
- Ignoring accessibility for visual-only information.
- Overdrawing expensive effects.
- Not checking dark mode and contrast.

## Senior iOS Engineer Perspective

Senior drawing design asks:

- Is this information or decoration?
- Can standard SwiftUI views do it?
- Would `Canvas` be more efficient?
- Does the animation respect reduce motion?
- Is the visual meaning accessible?
- Does it profile well on older devices?

## Interview Notes

Junior:

`Shape` draws paths. `Canvas` draws custom graphics.

Mid-level:

Use `TimelineView` for time-driven updates and `Canvas` for efficient custom drawing.

Senior:

I use SwiftUI drawing APIs when they reduce view complexity, profile animation cost, provide accessible alternatives, and avoid unnecessary timers or overdraw.

## Practice

1. Build a waveform with `Canvas`.
2. Build a custom badge shape.
3. Animate a simple drawing with `TimelineView`.
4. Explain when `Canvas` is better than many subviews.
