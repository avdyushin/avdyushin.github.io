---
title: Building a Clipboard Manager with SwiftUI for macOS
date: 2025-02-25T20:55:04+02:00
tags: [macos, swiftui]
---

In this post, I will show you how to build a simple clipboard manager for macOS using SwiftUI.

The app will display a list of the last items copied to the clipboard and allow you to copy them back to the clipboard.

![ClipMinder](/assets/img/clipminder/history.png)

# Introduction

I already have an app developed back in 2019 that does this, but it uses AppKit and I wanted to try building it with SwiftUI.

This time it will not be published on the App Store, but I will share the code on GitHub.

Having say that, I hope you enjoy this post and find it useful.

Please note, this is not a tutorial for beginners, but some findings I had while developing this app. Mostly for my future self.

## Status Bar item

The app will be a status bar item, so we need to create a new `MenuBarExtra` view and set menu view inside it.

```swift
MenuBarExtra("ClipMinder", systemImage: "paperclip") {
    AppMenu()
}
.menuBarExtraStyle(.menu)
```

As app runs in the background, we need to add a new key to the `Info` properties under the app's target to hide the dock icon: `Application is agent (UIElement)` to `YES`.

![Menu](/assets/img/clipminder/baritem.png)

## Settings

Show app settings is super easy using SwiftUI.
Just create a new view and place it inside a `Settings` view:

```swift
Settings {
    SettingsView()
}
```

![Settings](/assets/img/clipminder/settings.png)

## The Clipboard

To access the system clipboard, we need to use the `NSPasteboard` class.
It has `string(forType:)` and `setString(_:forType:)` methods to get and set the clipboard content.
Also it has a `changeCount` property to check if the clipboard content has changed.

Collecting pasteboard items using timer:

```swift
@Observable
final class PasteboardServiceImpl {

    private let pasteboard: NSPasteboard
    private var timer: AnyCancellable?
    private var changeCount = -1
    var currentItem: String? = nil

    init(polling: TimeInterval = 5, pasteboard: NSPasteboard = .general) {
        self.pasteboard = pasteboard
        changeCount = pasteboard.changeCount
        if let string = pasteboard.string(forType: .string) {
            currentItem = string
        }
        timer = Timer.publish(every: polling, on: .current, in: .default)
            .autoconnect()
            .compactMap { [weak self] _ in
                guard pasteboard.changeCount != self?.changeCount else {
                    return nil
                }
                guard pasteboard.availableType(from: [.string]) == .string else {
                    return nil
                }
                guard let string = pasteboard.string(forType: .string) else {
                    return nil
                }
                self?.changeCount = pasteboard.changeCount
                return string
            }
            .receive(on: DispatchQueue.main)
            .sink { [weak self] item in
                self?.currentItem = item
            }
    }

    deinit {
        timer?.cancel()
    }
```

Copying string back to the system clipboard:

```swift
pasteboard.declareTypes([.string], owner: self)
pasteboard.setString(string, forType: .string)
```

### Launch at login

To register app to launch at login, we need to use the `ServiceManagement` framework. It has a `SMAppService.mainApp` object to register and unregister the app.

```swift
final class AppLaunchServiceImpl: AppLaunchService {

    private var status: SMAppService.Status { SMAppService.mainApp.status }
    var launchAtLogin: Bool { status == .enabled }

    func toggleLaunchAtLogin(isLaunchAtLogin: Bool) {
        if isLaunchAtLogin {
            try? SMAppService.mainApp.register()
        } else {
            try? SMAppService.mainApp.unregister()
        }
    }
}
```

### Listen for local keystrokes

To listen for local keystrokes, we need to use the `NSEvent` class. It has a `addLocalMonitorForEvents(matching:handler:)` method to listen for local events.

We are interested in the `.keyDown` and `.flagsChanged` events only.

```swift
@Observable
final class LocalKeyListener {
    var hotKeyEvent = NSEvent()
    var isActive = false
    init() {
        NSEvent.addLocalMonitorForEvents(matching: [.keyDown]) { [weak self] event in
            guard let self else { return event }
            if isActive && !event.modifierFlags.toEventModifiers().isEmpty {
                hotKeyEvent = event
                isActive = false
            }
            return event
        }
        NSEvent.addLocalMonitorForEvents(matching: [.flagsChanged]) { [weak self] event in
            guard let self else { return event }
            if event.modifierFlags.toEventModifiers().isEmpty {
                isActive = true
            }
            return event
        }
    }
}
```

### Listen for global keystrokes

To listen for global keystrokes, we need to use the `CGEvent` class. It has a `tapCreate(tap:place:options:eventsOfInterest:callback:userInfo:)` method to listen for global events.

```swift
@Observable
final class GlobalKeyListener {

    private var eventTap: CFMachPort?
    var hotKeyTriggered = false

    init() {
        let callback: CGEventTapCallBack = { (proxy, type, event, info) -> Unmanaged<CGEvent>? in
            guard let rawValue = UserDefaults.standard.string(forKey: "hot_key") else {
                debugPrint("No hot key is defined, check settings!")
                return .passUnretained(event)
            }
            let hotKey = HotKey(rawValue: rawValue)
            let keyCode = event.getIntegerValueField(.keyboardEventKeycode)
            let modifiers = event.flags.toEventModifiers()
            if keyCode == hotKey.keyCode, modifiers == hotKey.modifiers {
                if let info {
                    Unmanaged<GlobalKeyListener>.fromOpaque(info).takeUnretainedValue().hotKeyTriggered = true
                    return nil
                }
            }
            return .passUnretained(event)
        }
        let userInfo = Unmanaged.passRetained(self).toOpaque()
        eventTap = CGEvent.tapCreate(
            tap: .cgSessionEventTap,
            place: .headInsertEventTap,
            options: .listenOnly,
            eventsOfInterest: (CGEventMask(1 << CGEventType.keyDown.rawValue)),
            callback: callback,
            userInfo: userInfo
        )
        guard let eventTap else {
            Unmanaged<GlobalKeyListener>.fromOpaque(userInfo).release()
            debugPrint("Can't create <CGEvent.tapCreate>!")
            return
        }
        let runLoop = CFMachPortCreateRunLoopSource(kCFAllocatorDefault, eventTap, 0)
        CFRunLoopAddSource(CFRunLoopGetCurrent(), runLoop, .defaultMode)
    }

    deinit {
        Unmanaged.passUnretained(self).release()
    }
}
```

### Conclusion

Launch app at login and keep it in status bar is a good way to have a clipboard manager always available.

Listening for global keystrokes is a good way to trigger the app without using the mouse.

### Source code

Full source code can be found at GitHub repo [ClipMinder](https://github.com/avdyushin/ClipMinder).
