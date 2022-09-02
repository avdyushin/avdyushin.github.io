---
title: Building Split View in SwiftUI
date: 2020-05-23T13:00:53+02:00
tags: [swift, swiftui]
---

## Introduction

### SwiftUI in Playground

Xcode's Playground templates unfortunately has none of them for SwiftUI.
For now we can create a new Blank Playground project and add boilerplate code:

```swift
import SwiftUI
import PlaygroundSupport

struct SplitView: View {
    var body: some View {
        Text("Split View")
    }
}

PlaygroundPage.current.setLiveView(SplitView())
```

Playground is ready, so once it run it will display live view preview.

### Creating new View

View Composition in SwiftUI:

```swift
struct SplitView<Content: View>: View {
    let content: () -> Content
    var body: some View {
        content()
    }
}
```

And set live view with new content:

```swift
PlaygroundPage.current.setLiveView(
    SplitView {
        Text("Split View")
    }
)
```

Let's place two views inside `VStack`:

```swift
struct SplitView<Content: View>: View {
    let topContent: () -> Content
    let bottomContent: () -> Content
    var body: some View {
        VStack {
            topContent()
            bottomContent()
        }
    }
}
```

Now we have two view split vertically:

```swift
PlaygroundPage.current.setLiveView(
    SplitView(
        topContent: {
            Text("Top View")
        },
        bottomContent: {
            Text("Bottom View")
        }
    )
)
```

What is we place `HStack` into bottom view?

```swift
HStack {
    Text("Bottom Left View")
    Text("Bottom Right View")
}
```

It will not work because now top and bottom content has different view types: `Text` and `HStack`.
To fix it we have to use two generic types to build out view:

```swift
struct SplitView<TopContent: View, BottomContent: View>: View {
    let topContent: () -> TopContent
    let bottomContent: () -> BottomContent
    var body: some View {
        VStack {
            topContent()
            bottomContent()
        }
    }
}
```

### Using @ViewBuilder

Complete custom view container using `@ViewBuilder`:

```swift
struct SplitView<TopContent: View, BottomContent: View>: View {
    let topContent: TopContent
    let bottomContent: BottomContent

    init(@ViewBuilder _ topContent: () -> TopContent, @ViewBuilder _ bottomContent: () -> BottomContent) {
        self.topContent = topContent()
        self.bottomContent = bottomContent()
    }

    var body: some View {
        VStack {
            topContent
            bottomContent
        }
    }
}

PlaygroundPage.current.setLiveView(
    SplitView({
        Text("Top View")
        Text("Top Title")
    },
    {
        HStack {
            Text("Bottom Left View")
            Text("Bottom Right View")
        }
    })
)
```

### SliderControlViewModel

To keep track of slider positions let's introduce view model:

```swift
class SliderControlViewModel: ObservableObject {
    @Published var current: CGFloat = 0 // 1
    @Published var previous: CGFloat = 0 // 2
}
```

1. Current relative position of the slider
1. Previous position

### GeometryReader

`GeometryReader` allows us to get size (and coordinates) of views.
We could use it to make a view have full width of all available space.

```swift
var body: some View {
    GeomertyReader { geometry in
        Text("View").frame(width: geometry.size.width)
    }
}
```

That `geometry` parameter also contains safe area insets.

### DragGesture

In SwiftUI we can attach custom gesture to any view.
We will attach `DragGesture` to slider control view
so that it can moved around.

### SliderControlView

```swift
struct SliderControl<Content: View>: View {

    @ObservedObject var viewModel: SliderControlViewModel

    var geometry: GeometryProxy // 1
    let content: Content

    init(
        viewModel: SliderControlViewModel,
        geometry: GeometryProxy,
        @ViewBuilder content: () -> Content) {
        self.viewModel = viewModel
        self.content = content()
        self.geometry = geometry
    }

    var body: some View {
        VStack { content }
        .offset(y: geometry.size.height / 2 + viewModel.current)
        .gesture(
            DragGesture() // 2
                .onChanged(onDragChanged)
                .onEnded(onDragEnded)
        )
    }

    fileprivate var maxLimit: CGFloat {
        geometry.size.height * 0.8
    }

    fileprivate var minLimit: CGFloat {
        geometry.size.height * 0.2
    }

    fileprivate func onDragChanged(_ gesture: DragGesture.Value) {
        let height = viewModel.previous + gesture.translation.height
        viewModel.current = max(maxLimit, min(minLimit, height)) // 3
    }

    fileprivate func onDragEnded(_ gesture: DragGesture.Value) {
        viewModel.previous = viewModel.current // 4
    }
}
```

1. Pass GeometryProxy to get hosting view size
1. Add DragGesture
1. Limit offset into min and max possible values
1. Save previous position

### SplitView

All things together:

```swift
struct SplitView<ControlView: View, TopContent: View, BottomContent: View>: View {

    @ObservedObject var viewModel: SliderControlViewModel

    var controlView: ControlView
    var topView: TopContent
    var bottomView: BottomContent

    init(
        viewModel: SliderControlViewModel,
        @ViewBuilder controlView: () -> ControlView,
        @ViewBuilder topView: () -> TopContent,
        @ViewBuilder bottomView: () -> BottomContent) {
        self.viewModel = viewModel
        self.controlView = controlView()
        self.topView = topView()
        self.bottomView = bottomView()
    }

    var body: some View {
        GeometryReader { geometry in
            ZStack {
                VStack {
                    Group {
                        self.topView
                            .frame(minWidth: 0, maxWidth: .infinity, minHeight: 0, maxHeight: .infinity)
                    }
                    Group {
                        self.bottomView
                            .frame(minWidth: 0, maxWidth: .infinity, minHeight: 0, maxHeight: .infinity)
                            .frame(height: geometry.size.height / 2 - self.viewModel.current)
                    }
                }
                SliderControl(viewModel: self.viewModel, geometry: geometry) {
                    Group {
                        self.controlView
                    }
                }
            } // ZStack
        } // GeometryReader
    }
}
```

### Links

- Swift Package [SplitView](https://github.com/avdyushin/SplitView)
