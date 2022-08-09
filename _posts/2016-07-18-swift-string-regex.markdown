---
title: String pattern matching in Swift
date: 2016-07-18 12:06:56
tags: ["Swift"]
---

Our goal is to use string pattern matching in easy-swifty way:

```swift
// Credit card number
let string = "4916474932438684"

// Check if it's Visa
if string =~ "^4[0-9]{6,}$" {
    /* */
}
```

<!-- more -->

Pattern matching with regular expression is supported by `range` function with `.regularExpression` options flag:

```swift
if string.range(of: "^4[0-9]{6,}$", options: [.regularExpression]) {
    // string is matches /^4[0-9]{6,}$/
}
```

Let's wrap this call into String's extension helper function called `matches`:

```swift
/// String extensions
extension String {
    /// Returns true if `String` matches regex `pattern`.
    /// - parameter pattern: pattern to search
    func matches(_ pattern: String) -> Bool {
        range(of: pattern, options: [.regularExpression]) != nil
    }
}
```

Then we already can shortify matching call with our `matches` function:

```swift
if string.matches("^4[0-9]{6,}$") {
    /* */
}
```

The last step is to create infix operator for our function:

```swift
/// String regex pattern matching operator
infix operator =~

/// Returns true if `string` matches regular expression `pattern`.
/// - parameter string: string to test
/// - parameter pattern: pattern to match
func =~ (string: String, pattern: String) -> Bool {
    string.matches(pattern)
}

// Usage:
if "cat" =~ "[cb]at" {
    /* */
}
```
