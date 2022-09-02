---
title: CALayer Animations
date: 2018-09-18 10:28:21+01:00
tags: [swift]
---

Create custom progress view using `CALayer` and `CoreAnimation`.

<!-- more -->

Start with base class and protocols.
Our base class should have `progress` property marked as `@NSManaged` to work with CoreAnimation.
Also we need to redraw our layer on progress property update.
To do it override `needsDisplay(forKey:)` method.

```swift
/// Base progress layer class
class BaseProgressLayer: CALayer {

    /// Progress key path constant string
    enum Keys: String {
        case progress = "progress"
    }

    /// To work with CoreAnimation this property should be marked ass @NSManaged
    /// which generates getter and setter
    @NSManaged var progress: CGFloat

    /// Redraw during progress value animation
    override class func needsDisplay(forKey key: String) -> Bool {
        if key == Keys.progress.rawValue {
            return true
        }
        return super.needsDisplay(forKey: key)
    }
}
```

Progress view protocol is simple. Just read-only `progress` value and `update` method to set new progress value.

```swift
/// View with progress value and ability to update
protocol ProgressableView {
    var progress: CGFloat { get }
    func update(_ progress: CGFloat, animated: Bool)
}
```

Now we can start with our progress view itself.
It will be generic class with `BaseProgressLayer` as `Layer` and confirmed to `ProgressableView` protocol.

```swift
/// Progress view itself which supports animated progress value changes
class ProgressView<Layer: BaseProgressLayer>: UIView, ProgressableView {

    /// This view is backed by our Layer
    override class var layerClass: AnyClass {
        return Layer.self
    }

    /// Update content scale to window's one
    override func didMoveToWindow() {
        super.didMoveToWindow()

        if let window = window {
            progressLayer.contentsScale = window.screen.scale
            progressLayer.setNeedsDisplay()
        }
    }

    /// Just for easy access
    var progressLayer: Layer {
        return self.layer as! Layer
    }

    public var progress: CGFloat {
        return progressLayer.progress
    }

    public func update(_ progress: CGFloat, animated: Bool) {
        // Not implemented yet
    }
}
```

Set new progress value without animation is very straightforward.
Remove progress animation, set new progress and set needs display.

```swift
private func updateInstantly(_ progress: CGFloat) {
    progressLayer.removeAnimation(forKey: BaseProgressLayer.Keys.progress.rawValue)
    progressLayer.progress = progress
    progressLayer.setNeedsDisplay()
}
```

To make it animated we need to create new `CABasicAnimation` for our `@NSManaged` progress key.
Also we need to keep new progress value after animation ended.

```swift
private func updateAnimated(_ progress: CGFloat) {
    progressLayer.removeAnimation(forKey: BaseProgressLayer.Keys.progress.rawValue)
    let animation = CABasicAnimation(keyPath: BaseProgressLayer.Keys.progress.rawValue)
    let oldValue = progressLayer.presentation()?.progress ?? 0
    progressLayer.progress = oldValue
    animation.fromValue = oldValue
    animation.toValue = progress
    animation.duration = CFTimeInterval(fabsf(Float(oldValue - progress)))
    animation.fillMode = kCAFillModeForwards
    animation.isRemovedOnCompletion = true
    animation.timingFunction = CAMediaTimingFunction(name: kCAMediaTimingFunctionEaseInEaseOut)
    animation.delegate = self
    progressLayer.add(animation, forKey: BaseProgressLayer.Keys.progress.rawValue)
}

func animationDidStop(_ anim: CAAnimation, finished flag: Bool) {
    if let value = anim.value(forKey: "toValue") as? CGFloat {
        progressLayer.progress = value
    }
}
```

Now we can complete `update` method:

```swift
public func update(_ progress: CGFloat, animated: Bool) {
    if animated {
        updateAnimated(progress)
    } else {
        updateInstantly(progress)
    }
}
```

We have flexible progress view which draws nothing!
It's time to make simple circle progress view.
First of all we need to create new layer subclass to make our drawings of progress.

We override `draw(in:)` function to draw our filled circles based on progress value.

```swift
/// Layer to style circle view progress
class CircleProgressLayer: BaseProgressLayer {

    @NSManaged var segmentsCount: Int
    @NSManaged var trackColor: UIColor
    @NSManaged var trackLineWidth: CGFloat
    @NSManaged var progressColor: UIColor
    @NSManaged var progressLineWidth: CGFloat

    override func draw(in ctx: CGContext) {
        ctx.clear(self.bounds)
        drawTrack(in: ctx)
        drawProgress(in: ctx)
    }

    private var step: CGFloat {
        return CGFloat.pi * 2 / CGFloat(segmentsCount * 2)
    }

    private var radius: CGFloat {
        return min(self.bounds.width, self.bounds.height) / 2 - max(trackLineWidth, progressLineWidth) / 2
    }

    private var center: CGPoint {
        return CGPoint(x: self.bounds.midX, y: self.bounds.midY)
    }

    private func drawTrack(in ctx: CGContext) {
        ctx.setLineWidth(trackLineWidth)
        ctx.setStrokeColor(trackColor.cgColor)
        drawSegmentes(in: ctx, progress: 1.0)
    }

    private func drawProgress(in ctx: CGContext) {
        ctx.setLineWidth(progressLineWidth)
        ctx.setStrokeColor(progressColor.cgColor)
        drawSegmentes(in: ctx, progress: progress)
    }

    private func drawSegmentes(in ctx: CGContext, progress: CGFloat) {
        if segmentsCount <= 1 {
            let circle = UIBezierPath(
                arcCenter: center,
                radius: radius,
                startAngle: -CGFloat.pi / 2,
                endAngle: (progress * CGFloat.pi * 2) - CGFloat.pi / 2,
                clockwise: true
            )
            ctx.setLineCap(.round)
            ctx.addPath(circle.cgPath)
            ctx.strokePath()
        } else {
            let count = Int(CGFloat(segmentsCount) * progress)
            var current = -CGFloat.pi / 2
            for _ in 0..<count {
                let arc = UIBezierPath(
                    arcCenter: center,
                    radius: radius,
                    startAngle: current,
                    endAngle: current + step,
                    clockwise: true
                )
                ctx.setLineCap(.square)
                ctx.addPath(arc.cgPath)
                ctx.strokePath()
                current += step * 2
            }
        }
    }
}
```

Appearance of the progress like track and progress colors can be set during view initialization.
Our `CircleProgressView` is just `ProgressView<CircleProgressLayer>`.
All extra code is only to set up appearance properties.

```swift
/// Circle progress view width custom options
class CircleProgressView: ProgressView<CircleProgressLayer> {

    struct Options {
        let segmentsCount: Int
        let trackColor: UIColor
        let trackLineWidth: CGFloat
        let progressColor: UIColor
        let progressLineWidth: CGFloat

        init(segmentsCount: Int = 1,
             trackColor: UIColor = UIColor(red: 0, green: 148/255.0, blue: 50/255.0, alpha: 1.0),
             trackLineWidth: CGFloat = 8,
             progressColor: UIColor = UIColor(red: 196/255.0, green: 229/255.0, blue: 56/255.0, alpha: 1.0),
             progressLineWidth: CGFloat = 6) {

            self.segmentsCount = segmentsCount
            self.trackColor = trackColor
            self.trackLineWidth = trackLineWidth
            self.progressColor = progressColor
            self.progressLineWidth = progressLineWidth
        }
    }

    var options: Options = Options() {
        didSet {
            applyOptions()
        }
    }

    convenience init(options: Options) {
        self.init()

        self.options = options
        applyOptions()
    }

    private func applyOptions() {
        progressLayer.segmentsCount = options.segmentsCount
        progressLayer.trackColor = options.trackColor
        progressLayer.trackLineWidth = options.trackLineWidth
        progressLayer.progressColor = options.progressColor
        progressLayer.progressLineWidth = options.progressLineWidth
        progressLayer.setNeedsDisplay()
    }
}
```

Full circle:

![](/posts/calayer-animations/circle.png)

Segmented one:

![](/posts/calayer-animations/segmented-circle.png)

The same with line progress view.
Layer:

```swift
/// Layer to style a line view progress
class LineProgressLayer: BaseProgressLayer {

    @NSManaged var segmentsCount: Int
    @NSManaged var lineWidth: CGFloat
    @NSManaged var spacing: CGFloat
    @NSManaged var trackColor: UIColor
    @NSManaged var progressColor: UIColor

    override func draw(in ctx: CGContext) {
        ctx.clear(self.bounds)
        drawTrack(in: ctx)
        drawProgress(in: ctx)
    }

    private var segmentWidth: CGFloat {
        return (self.bounds.width - CGFloat(segmentsCount - 1) * spacing) / CGFloat(segmentsCount)
    }

    private var centerY: CGFloat {
        return self.bounds.midY - lineWidth / 2
    }

    private func drawTrack(in ctx: CGContext) {
        ctx.setFillColor(trackColor.cgColor)
        drawSegments(in: ctx, progress: 1.0)
    }

    private func drawProgress(in ctx: CGContext) {
        ctx.setFillColor(progressColor.cgColor)
        drawSegments(in: ctx, progress: progress)
    }

    private func drawSegments(in ctx: CGContext, progress: CGFloat) {
        if segmentsCount <= 1 {
            let rect = UIBezierPath(roundedRect: CGRect(
                x: -lineWidth / 2,
                y: centerY,
                width: (bounds.width + lineWidth) * progress,
                height: lineWidth
            ), cornerRadius: lineWidth / 2)
            ctx.addPath(rect.cgPath)
            ctx.fillPath()
        } else {
            let count = Int(CGFloat(segmentsCount) * progress)
            for i in 0..<count {
                ctx.fill(CGRect(
                    x: CGFloat(i) * (segmentWidth + spacing),
                    y: centerY,
                    width: segmentWidth,
                    height: lineWidth
                ))
            }
        }
    }
}
```

View:

```swift
/// Line progress view with custom options
class LineProgressView: ProgressView<LineProgressLayer> {

    struct Options {
        let segmentsCount: Int
        let spacing: CGFloat
        let lineWidth: CGFloat
        let trackColor: UIColor
        let progressColor: UIColor

        init(segmentsCount: Int = 1,
             spacing: CGFloat = 4,
             lineWidth: CGFloat = 8,
             trackColor: UIColor = UIColor(red: 0, green: 98/255.0, blue: 102/255.0, alpha: 1.0),
             progressColor: UIColor = UIColor(red: 18/255.0, green: 203/255.0, blue: 196/255.0, alpha: 1.0)) {

            self.segmentsCount = segmentsCount
            self.spacing = spacing
            self.lineWidth = lineWidth
            self.trackColor = trackColor
            self.progressColor = progressColor
        }
    }

    var options: Options = Options() {
        didSet {
            applyOptions()
        }
    }

    convenience init(options: Options) {
        self.init()

        self.options = options
        applyOptions()
    }

    private func applyOptions() {
        progressLayer.segmentsCount = options.segmentsCount
        progressLayer.spacing = options.spacing
        progressLayer.lineWidth = options.lineWidth
        progressLayer.trackColor = options.trackColor
        progressLayer.progressColor = options.progressColor
        progressLayer.setNeedsDisplay()
    }
}
```

Single line:

![](/posts/calayer-animations/line.png)

With segments:

![](/posts/calayer-animations/segmented-line.png)

Source and Xcode playground:

[https://github.com/avdyushin/ProgressView](https://github.com/avdyushin/ProgressView)
