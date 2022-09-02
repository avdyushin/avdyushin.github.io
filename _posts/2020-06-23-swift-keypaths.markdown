---
title: Swift keypaths
date: 2020-06-23T11:10:41+02:00
tags: [swift]
---

Extending `Sequence` to have method that calculates sum:

```swift
extension Sequence where Element: AdditiveArithmetic {
    func sum() -> Element { reduce(.zero, +) }
}
```

This `sum()` available for any sequence with values that confirms `AdditiveArithmetic` protocol.

Consider we have custom struct like and array of this objects:

```swift
struct LocationData {
    let speed: Double
}

let data = [10, 11, 14, 15, 19].map(LocationData.init)
```

To calculate average speed we find sum and divide by count:

```swift
let avgSpeed = data.map { $0.speed }.sum() / Double(data.count)
// avgSpeed = 13.8
```

Using keypath we can reference to properties:

```swift
let speedKeyPath = \LocationData.speed
```

Let's extend sequence to have method that calculates sum of given keypath element:

```swift
extension Sequence {
    func sum<T: AdditiveArithmetic>(by keyPath: KeyPath<Element, T>) -> T {
        map { $0[keyPath: keyPath] }.sum()
    }
}
```

Now average speed calculation can be expressed shorter:

```swift
let avgSpeed = data.sum(by: \.speed) / Double(data.count)
// avgSpeed = 13.8
```
