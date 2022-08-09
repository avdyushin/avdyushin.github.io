---
title: "Swift DI using Property Wrappers"
date: 2020-05-14T16:53:45+02:00
---

# Dependency Injection using Swift Property Wrappers

Note: Code tested using Swift 5.2.2 and Xcode 11.4.1

## Dependency Injection

Dependency Injection (DI) helps us to separate creation and usage of an object.
We can replace implementations (in Unit Tests for example) without changing the object that uses dependencies.

<!-- more -->

In example below `Service` is tightly coupled with `ViewModel` object:

```swift
class Service { }
class ViewModel {
    let service: Service
    init() {
        self.service = Service() // ViewModel knows how to create Service
    }
}
```

We can use property injection DI, so `ViewModel` will not create `Service` by itself:

```swift
class MockService: Service { }
class ViewModel {
    let service: Service
    init(service: Service) {
        self.service = service // Can be any subclass of Service
    }
}
```

Using protocols helps us to hide concrete types, so ViewModel should know only how to use object.
Our goal is to use DI with Swift's property wrapper like this:

```swift
protocol ServiceProtocol { }
class Service: ServiceProtocol { }
class MockService: ServiceProtocol { }
class ViewModel {
    @Injected var service: ServiceProtocol
    init() {
        service.start() // ViewModel can start using service
    }
}
```

In order to do achieve this we need 3 main components:

1. Dependency resolver;
1. Dependency container;
1. Custom property wrapper.

### Dependency Resolver

Dependency resolver is a simple factory.
We provide block which returns an object we want to inject into dependency container.

```swift
struct Dependency {
    typealias ResolveBlock<T> = () -> T

    private(set) var value: Any! // Actual value will be assigned after resolve() call
    private let resolveBlock: ResolveBlock<Any>
    let name: String

    init<T>(_ block: @escaping ResolveBlock<T>) {
        resolveBlock = block // Save block for future
        name = String(describing: T.self)
    }
    mutating func resolve() {
        value = resolveBlock()
    }
}
```

Actual injection will be performed after `resolve()` call.

```swift
let service = Dependency { Service() }
service.value // nil
service.resolve()
service.value // Service instance
```

### Dependency Container

Dependency container will manage added `Depenedency` objects.

1. Register given `Dependency` in container;
1. Build resolved dependencies list;
1. Resolve single dependency of given type.

```swift
class Dependencies {

    static private(set) var shared = Dependencies() // 1

    fileprivate var dependencies = [Dependency]() // 2

    func register(_ dependency: Dependency) {
        // Avoid duplicates
        guard dependencies.firstIndex(where: { $0.name == dependency.name }) == nil else {
            debugPrint("\(String(describing: dependency.name)) already registered, ignoring")
            return
        }
        dependencies.append(dependency)
    }

    func build() {
        // We assuming that at this point all needed dependencies are registered
        for index in dependencies.startIndex..<dependencies.endIndex {
            dependencies[index].resolve()
        }
        Self.shared = self // 3
    }

    func resolve<T>() -> T {
        guard let dependency = dependencies.first(where: { $0.value is T })?.value as? T else {
            fatalError("Can't resolve \(T.self)")
        }
        return dependency
    }
}
```

1. Main access point to the container;
1. List of available dependencies to be resolved and accessed;
1. Once initial resolve is complete we have to update shared value.

Now we can setup simple dependency container with injected service:

```swift
protocol LocationService { /* start() */ }
protocol JourneyService { /* start() */ }
class MockLocation: LocationService { /* start() */ }
class MockJourney: JourneyService { /* start() */ }

let location = Dependency { MockLocation() } // Future injection of LocationService
let journey = Dependency { MockJourney() } // Future injection of JourneyService
let dependencies = Dependencies()
dependencies.register(location)
dependencies.register(journey)
```

At this point we have stored only blocks to be resolved.
Deferred call to `build()` will finish setup:

```swift
dependencies.build() // Resolve
```

And usage inside `ViewModel`:

```swift
class ViewModel {
    let service: LocationService = Dependencies.shared.resolve()
    init() {
        service.start() // Service is MockLocation instance
    }
}
```

It's time to start with wrapping access to `Dependencies.shared` via `@propertyWrapper` feature.

### Property Wrapper

Property wrapper it's a struct (class or enum) with defined `wrappedValue` property.
We can wrap dependency container calls into property wrapper:

```swift
@propertyWrapper
struct Injected<Dependency> {

    var dependency: Dependency! // Resolved dependency

    var wrappedValue: Dependency {
        mutating get {
            if dependency == nil {
                let copy: Dependency = Dependencies.shared.resolve()
                self.dependency = copy // Keep copy
            }
            return dependency
        }
        mutating set {
            dependency = newValue
        }
    }
}
```

So now we can apply property wrapper in this way:

```swift
@Injected var service: LocationService
```

### Using Swift DSL for container

Adding `@_functionBuilder` struct and convenience initializers into `Dependencies` class will make it more Swifty:

```swift
class Dependencies {
    @_functionBuilder struct DependencyBuilder {
        static func buildBlock(_ dependency: Dependency) -> Dependency { dependency }
        static func buildBlock(_ dependencies: Dependency...) -> [Dependency] { dependencies }
    }

    convenience init(@DependencyBuilder _ dependencies: () -> [Dependency]) {
        self.init()
        dependencies().forEach { register($0) }
    }

    convenience init(@DependencyBuilder _ dependency: () -> Dependency) {
        self.init()
        register(dependency())
    }

    /* Previous code */
}
```

Usage:

```swift
let dependencies = Dependencies {
    Dependency { LocationImpl() }
    Dependency { JourneyImpl() }
    // ...
}
dependencies.build()
```

### Iterate over injected dependencies

If we need to manipulate on set of injected dependencies we can make dependency container
conform to `Sequence` protocol:

```swift
extension Dependencies: Sequence {
    func makeIterator() -> AnyIterator<Any> {
        var iter = dependencies.makeIterator()
        return AnyIterator { iter.next()?.value }
    }
}
```

Now it's easy to find all dependencies of given protocol and do what we want:

```swift
protocol Resettable { func reset() }
Dependencies.shared
    .compactMap { $0 as? Resettable }
    .forEach { $0.reset() }
```

## Full Example

### Service protocols

```swift
protocol LocationService {
    var location: AnyPublisher<CLLocation, Never> { get }
    func start()
}
protocol JourneyService {
    func start()
}
```

#### Location service implementations

```swift
import Combine
import Foundation
import CoreLocation

class RealLocation: LocationService {
    var location: AnyPublisher<CLLocation, Never>

    init() { /* setup location service and connect publisher */ }
    func start() {
        debugPrint("Real Location service has been started")
    }
}

class MockLocation: LocationService {

    private let timer = Timer.publish(every: 1, on: RunLoop.main, in: .default) // 1
    private let subject = PassthroughSubject<CLLocation, Never>() // 2
    private var cancellables = Set<AnyCancellable>()

    var location: AnyPublisher<CLLocation, Never>

    init() {
        location = subject.eraseToAnyPublisher()
        timer
            .map { _ in
                CLLocation(
                    latitude: CLLocationDegrees.random(in: 50..<55),
                    longitude: CLLocationDegrees.random(in: 33..<36)
                )
            }
            .subscribe(subject) // 3
            .store(in: &cancellables)
    }

    func start() {
        timer
            .connect() // 4
            .store(in: &cancellables)
        debugPrint("Mock Location service has been started")
    }
}
```

1. Create Timer publisher to send values to pipeline each second on main RunLoop;
1. Private publisher to forward location into public;
1. Connect subscriber (timer will not start at this point);
1. Start timer.

#### Journey service implementations

Just simple Journey service implementations for testing:

```swift
class RealJourney: JourneyService {
    func start() {
        debugPrint("Real Journey service has been started!")
    }
}
class MockJourney: JourneyService {
    func start() {
        debugPrint("Mock Journey service has been started!")
    }
}
```

### Building dependencies

```swift
let dependencies = Dependencies {
    Dependency { RealLocation() } // Register LocationService
    Dependency { RealJourney() } // Register JourneyService
}
// Resolve only when it's needed
dependencies.build()
```

### Usage

Simple Journey view model:

```swift
class JourneyViewModel {

    @Injected private var location: LocationService
    @Injected private var journey: JourneyService

    private var cancellables = Set<AnyCancellable>()

    func start() {
        location.start()
        location.location
            .sink { debugPrint($0) }
            .store(in: &cancellables)
    }

    func ride() {
        journey.start()
    }
}
```

Start receiving locations:

```swift
let viewModel = JourneyViewModel()
viewModel.start() // location is resolved with RealLocation
```

Will output 'Real Location service has been started'.
Because injected property is resolved on the first access to it we can replace implementation by rebuilding dependencies container:

```swift
Dependencies {
    Dependency { MockLocation() }
    Dependency { MockJourney() }
}.build()

// viewModel.location is still RealLocation
viewModel.ride() // but journey is resolved with MockJourney
```

Will output 'Mock Journey service has been started'.

## Links

- Based on [this](https://basememara.com/swift-dependency-injection-via-property-wrapper/)
- Similar [article](https://medium.com/better-programming/taking-swift-dependency-injection-to-the-next-level-b71114c6a9c6)
- [Property Wrappers](https://docs.swift.org/swift-book/LanguageGuide/Properties.html#ID617)
- [Combine](https://developer.apple.com/documentation/combine)
