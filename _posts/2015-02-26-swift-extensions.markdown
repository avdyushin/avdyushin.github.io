---
title: Extensions in Swift
date: 2015-02-26 14:19:26
tags: [swift]
---

Extensions in _Swift_ allow us to add methods to existing objects (`class`, `struct`, `enum` and `protocol`).
They are similar to categories in _Objective-C_.

Let's extend `UIColor` class with custom convenience initialiser to create color with hex value:

```swift
extension UIColor {

    /// Returns color with given hex integer value
    convenience init(hex: Int, alpha: CGFloat = 1.0) {
        let r = CGFloat((hex & 0xff0000) >> 16) / 255.0
        let g = CGFloat((hex & 0x00ff00) >>  8) / 255.0
        let b = CGFloat((hex & 0x0000ff) >>  0) / 255.0

        self.init(red: r, green: g, blue: b, alpha: alpha)
    }
}
```

Now we can create colors in this way:

```swift
let color1 = UIColor(hex: 0x1abc9c, alpha: 0.5)
let color2 = UIColor(hex: 0x112233)
```

We can also add methods and read-only properties to instances of `UIColor` class:

```swift
extension UIColor {

    /// Returns hex string color representation
    func toHexString(prefix: String = "#") -> String {
        String(format:"\(prefix)%06x", asInt)
    }

    /// Returns integer color representation
    var asInt: Int {
        var r: CGFloat = 0, g: CGFloat = 0, b: CGFloat = 0, a: CGFloat = 0

        getRed(&r, green: &g, blue: &b, alpha: &a)

        return (Int)(r * 255) << 16 | (Int)(g * 255) << 8  | (Int)(b * 255)  << 0
    }
}
```

New methods `asInt` and `asHexString` now available for any `UIColor` instance:

```swift
let myRed = UIColor(hex: 0x991122) // r 0,6 g 0,067 b 0,067 a 1,0
let intValue = myRed.asInt // 10031377
let cssColor = myRed.toHexString() // "#991122"
```

Static (or class) methods and read-only properties can be defined in extensions as well:

```swift
extension UIColor {
    static var myRed: Color { UIColor(hex: 0xAA0000) }
}
```

Usage:

```swift
let myColor = UIColor.myRed
```

## Links
1. [Extensions](https://docs.swift.org/swift-book/LanguageGuide/Extensions.html)
1. [My UIColor extensions](https://github.com/avdyushin/FlatUIColor)
