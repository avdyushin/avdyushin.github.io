---
title: Using Mixins in Swift
date: 2017-03-19 20:28:21+01:00
tags: [swift]
---

Mixin — it is a protocol with default implementation of methods.

<!-- more -->

Let's say we have a registration screen with email input text field that needs to be validated to be sure it is correct email address.

We can implement it in this way:

```swift
class RegisterViewController: UIViewController {

    fileprivate func isValidEmail(_ email: String) -> Bool {
    ...
    }

    @IBOutlet var emailTextField: UITextField!
    @IBAction func registerButtonDidPress() {
        guard isValidEmail(emailTextField.text!) else {
            print("Incorrect email provided")
            return
        }
        ...
    }
}
```

We could put validation logic directly into view controller, but if we do that with everything then it will be too massive.

One way to improve this is to create extension for `String` class which can be reused in future.

```swift
extension String {
    var isValidEmail: Bool {
    ...
    }
}
```

Now validation logic is removed from view controller and can be reused.
Another alternative solution to improve our code is to use mixins.

```swift
protocol EmailValidatable {
    func isValidEmail(_ email: String) -> Bool
}
```

Now we can create extension on protocol and add default implementation of `isValidEmail` method.

```swift
extension EmailValidatable {
    func isValidEmail(_ email: String) -> Bool {
    ...
    }
}
```

Once our view controller confirms to `EmailValidatable` protocol, it has access to `isValidEmail` method.

```swift
class RegisterViewController: UIViewController, EmailValidatable {
    @IBOutlet var emailTextField: UITextField!
    @IBAction func registerButtonDidPress() {
        guard isValidEmail(emailTextField.text!) else {
            print("Incorrect email provided")
            return
        }
        ...
    }
}
```

We just avoided extra helper objects and added new protocol :)

Type-safe API

Let's create mixin `Reusable`:

```swift
protocol Reusable: class {
    static var reusableIdentifier: String { get }
}

extension Reusable {
    static var reusableIdentifier: String {
        return String(describing: Self.self)
    }
}
```

Define new cell class which confirms to `Reusable` protocol:

```swift
class MyCell: UITableViewCell, Reusable {
    func updateWithData(_ data: Objects) {
    ...
    }
}
```

Now we can extend `UITableView` to dequeue cells in easy way:

```swift
extension UITableView {
    func dequeueReusableCell<T>(for indexPath: IndexPath) -> T where T: UITableViewCell, T: Reusable {
        return self.dequeueReusableCell(withIdentifier: T.reusableIdentifier, for: indexPath) as! T
    }
}
```

Using new function we never reusable identifier as strings, so we can't make typos.

```swift
let cell: MyCell = tableView.dequeueReusableCell(for: indexPath)
cell.updateWithData(someData)
```

Links:

<a href="https://alisoftware.github.io/swift/protocol/2015/11/08/mixins-over-inheritance" target="_blank">https://alisoftware.github.io/swift/protocol/2015/11/08/mixins-over-inheritance</a>
<a href="https://github.com/AliSoftware/Reusable" target="_blank">https://github.com/AliSoftware/Reusable</a>
