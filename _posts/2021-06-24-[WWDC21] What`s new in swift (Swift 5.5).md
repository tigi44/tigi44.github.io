---
title: "[WWDC21] What`s new in swift (Swift 5.5)"
excerpt: "[WWDC21] What`s new in swift (Swift 5.5)"
description: "[WWDC21] What`s new in swift (Swift 5.5)"
modified: 2021-06-24
categories: "WWDC21"
tags: [WWDC21, Swift, Swift5.5]

header:
  teaser: /assets/images/teaser/wwdc21-teaser.jpg
---

# Diversity
- [swift.org/diversity](swift.org/diversity){:target="_blank"}

# Update on Swift Package
## Package Collections in Xcode
- like the [swift package index](https://swiftpackageindex.com){:target="_blank"}
- You can now simply browse a collection and add packages from a new package search screen in Xcode
- [swift.org/blog/package-collections](swift.org/blog/package-collections){:target="_blank"}

![swift1](/assets/images/post/wwdc21/swift/swift1.png)
- simple JSON files you can publish anywhere
- curated lists of packages for different use cases

## Apple Packages
- Collections Package, Algorithms Package, System Package, Numeric Package, ArgumentParser Package, ...

### The Collections Package
- a new opensource package of data structure
- [github.com/apple/swift-collections](github.com/apple/swift-collections){:target="_blank"}

#### Deque
- an Array

```swift
import Collections

var colors: Deque = ["red", "yellow", "blue"]

colors.prepend["green"]
colors.append["orange"]
// colors is now ["green", "red", "yellow", "blue", "orange"]

colors.popFirst()
colors.popLast()
// colors is now ["red", "yellow", "blue"]
```

#### OrderedSet
- powerful hybrid of an Array and a Set

```swift
import Collections

var buildingMaterials: OrderdSet = ["straw", "sticks", "bricks"]

for i in 0 ..<buildingMaterials.count {
    print("#\(i) \(buildingMaterials[i]")
}

buildingMaterials.append("straw") // (inserted: false, index: 0)
```

#### OrderedDictionary
- a useful alternative to Dictionary when order is important

```swift
import Collections

var responses: OrderedDictionary = [200: "OK", 403: "Forbidden", 404: "Not Found"]

var (cod, phrase) in responses {
    print("\(code) (\(phrase))")
}
```

### The Algorithms Package
- Sequence and Collection algorithms
- [github.com/apple/swift-algorithms](github.com/apple/swift-algorithms){:target="_blank"}

```swift
import Algorithms

let testAccounts = [ ... ]

for testGroup in testAccounts.uniquePermutations(ofCount: 0...) {
    try validate(testGroup)
}

let randomGroup = testAccounts.randomSample(count: 5)
```

### The System Package
- a library providing idiomatic, low-level interfaces to system calls
- [github.com/apple/swift-system](github.com/apple/swift-system){:target="_blank"}

```swift
import System

var path: FilePath = "/tmp/WWDC2021.txt"
print(path.lastComponent)   // "WWDC2021.txt"
print(path.extension)       // "txt"
```

### The Numeric Package
- [github.com/apple/swift-numeric](github.com/apple/swift-numeric){:target="_blank"}

```swift
import Numerics

let x: Float16 = 1.5
let y = Float16.exp(x)

let z = Complex(0, Float16.pi)
let w = Complex.exp(z)
```

### The ArgumentParser Package
- [github.com/apple/swift-argument-parser](github.com/apple/swift-argument-parser){:target="_blank"}
- to generate code-completion scripts for the Fish shell
- joined short options
- improved error messages
- If you've used the Swift Package Manager command-line tool recently, you've used Swift ArgumentParser.

# Update on Swift on Server
- static linking on Linux
- improved JSON performance
- enhanced and optimized the performance of the AWS Lambda runtime

# Developer Experience improvements
## Swift DocC
- a documentation compiler that's deeply integrated inside Xcode 13

![swift2](/assets/images/post/wwdc21/swift/swift2.png)

## Performance improvements in the type checker
- sped up the performance for type checking of array literals

## Build Improvements
- Faster builds when changing imported modules
    - no longer rebuild every source file
- Faster startup time before launching compilers
- Fewer recompilations after changing an extension body

![swift3](/assets/images/post/wwdc21/swift/swift3.png)
![swift4](/assets/images/post/wwdc21/swift/swift4.png)

# Memory Management
- ARC automatically frees up the memory used by class instances when those instances are no longer needed

```swift
class Traveler {
    var destination: String
}

func test() {
    let traveler1 = Traveler(destination: "Unknown")
    // retain
    let traveler2 = traveler1
    // release
    traveler2.destination = "Big Sur"
    // release
    print("Done traveling")
}
```
![swift5](/assets/images/post/wwdc21/swift/swift5.png)

# Ergonomic improvements
## Result Builders
## Enum Codable synthesis
```swift
enum Command: Codable {
    case load(key: String)
    case store(key: String, value: Int)
}
```

## Flexible Static member lookup
- like enum, using the same dot-notation on Structures

```swift
protocol Coffee { ... }
struct RegualrCoffee: Coffee { }
struct Cappuccino: Coffee { }
extension Coffee where Self == Cappuccino {
    static var cappuccino: Cappuccino { Cappucino() }
}

func brew<CoffeeType: Coffee>(_ coffee: CoffeeType) { ... }

brew(.capuccino.large)
```

## Property wrappers on parameters

```swift
@propertyWrapper
struct NonEmpty<Value: Collection> {
    init(wrappedValue: Value) {
        precondition(!wrappedValue.isEmpty)
        self.wrappedValue = wrappedValue
    }

    var wrappedValue: Value {
        willSet { precondition(!newValue.isEmpty) }
    }
}

struct User {
    @NonEmpty var username: String
}

func logIn(@NonEmpty _ username: String) {
    print("\(username)")
}
```
## SwiftUI Code
```swift
import SwiftUI

struct SettingsView: View {
    @State var settings: [Setting]

    private let padding = 10.0

    var body: some View {
        List($settings) { $setting in
            Toggle(setting.displayName, isOn: $setting.isOn)
            #if os(masOS)
            .toggleStyle(.checkbox)
            #else
            .toggleStyle(.switch)
            #endif
        }
        .padding(padding)
    }
}
```

# Asynchronous and Concurrent programming
- the highlight of Swift 5.5

![swift6](/assets/images/post/wwdc21/swift/swift6.png)

## Asynchronous : Async/Await
```swift
func fetchImage(id: String) async throws -> UIImage {
    let request = self.imageURLRequest(for: id)
    let (data, response) = try await URLSession.shared.data(for: request)
    if let httpResponse = response as? HTTPURLResponse,
       httpResponse.statusCode != 200 {
        throw TransferFailure()
    }
    guard let image = UIImage(data: data) else {
        throw ImageDecodungFailure()
    }
    return image
}
```

## Structed Concurrency
- async
- it will be able to suspend if it needs to wait for results that are being computed in other threads

```swift
func titleImage() async throws -> Image {
    let background = try renderBackground()
    let foreground = try renderForeground()
    let title = try renserTitle()
    return merge(background, foreground, title)
}
```

- async let : the async let syntax to run the first two operations in parallel.

```swift
func titleImage() async throws -> Image {
    async let background = renderBackground()
    async let foreground = renderForeground()
    let title = try renserTitle()
    return try await merge(background, foreground, title)
}
```
![swift7](/assets/images/post/wwdc21/swift/swift7.png)

# Actor

- a newconstruct helps protect your data in a multi-threaded environment

```swift
// before : Concurrent code
class Statistics {
    private var counter: Int = 0
    func increment() {
        counter += 1
    }
}

// with swift actor
actor Statistics {
    private var counter: Int = 0
    func increment() {
        counter += 1
    }
}
var statistics = Statistics()
await statistics.increment()

// use async/await in actor
actor Statistics {
    private var counter: Int = 0
    func increment() {
        counter += 1
    }
    func publish() async {
        await sendResults(counter)
    }
}
```


# Reference
- [https://developer.apple.com/wwdc21/10192](https://developer.apple.com/wwdc21/10192){:target="_blank"}
