---
title: "[iOS, Swift] Dependency Injection using ObjectIdentifier"
excerpt: "the foundation of dependency injection using ObjectIdentifier in Swift"
description: "the foundation of dependency injection using ObjectIdentifier in Swift"
modified: 2022-04-05
categories: "iOS"
tags: [iOS, Swift, DependencyInjection, ObjectIdentifier]

toc: true

header:
  teaser: /assets/images/teaser/swift-teaser.png
---

# ObjectIdentifier
- [https://developer.apple.com/documentation/swift/objectidentifier](https://developer.apple.com/documentation/swift/objectidentifier){:target="_blank"}

```swift
class Person {
    let name: String

    init(name: String) {
        self.name = name
    }
}

let kim          = Person(name: "kim")
let jeon         = Person(name: "jeon")

let idOfPerson   = ObjectIdentifier(Person.self)
let idOfKim      = ObjectIdentifier(kim)
let idOfJeon     = ObjectIdentifier(jeon)

print("Person Type id: \(idOfPerson)")
print("kim id: \(idOfKim)")
print("jeon id: \(idOfJeon)")

// print
// Person Type id: ObjectIdentifier(0x00000001123f1320)
// kim id: ObjectIdentifier(0x00007f8d9506de10)
// jeon id: ObjectIdentifier(0x00007f8d950a6150)
```

# Dependency Injection using ObjectIdentifier

## Example SourceCode
```swift
protocol DataSource {
    func fetchData() -> String
}

class NetworkDataSource: DataSource {
    func fetchData() -> String {
        return "a string data from Network API"
    }
}

class Repository {
    private let dataSource: DataSource

    init(dataSource: DataSource) {
        self.dataSource = dataSource
    }

    func fetch() -> String {
        return self.dataSource.fetchData()
    }
}
```

## Dependency Injection
- Dependency Injection with Register/Resolve Pattern

```swift
class DIResolver {
    private var map: [ObjectIdentifier: AnyObject] = [:]

    func register(_ aClass: AnyObject) {
        let id = ObjectIdentifier(type(of: aClass))

        guard map[id] == nil else {
            return
        }

        map[id] = aClass
    }

    func resolve<T>(type: T.Type) -> T? {
        let id = ObjectIdentifier(type)

        return map[id] as? T
    }
}
```

```swift
let dataSourceResolver = DIResolver()

dataSourceResolver.register(NetworkDataSource())

if let dataSource = dataSourceResolver.resolve(type: NetworkDataSource.self) {
    let repository = Repository(dataSource: dataSource)
    print(repository.fetch())
}
```

## for Protocol Type
```swift
class DIResolver {
    ...

    func register<T>(_ aClass: T, for protocolType: T.Type) {
        let id = ObjectIdentifier(protocolType)

        guard map[id] == nil,
              let classObject = aClass as? AnyObject else {
            return
        }

        map[id] = classObject
    }

    ...
}
```

```swift
let dataSourceResolver = DIResolver()

dataSourceResolver.register(NetworkDataSource(), for: DataSource.self)

if let dataSource = dataSourceResolver.resolve(type: DataSource.self) {
    let repository = Repository(dataSource: dataSource)
    print(repository.fetch())
}

```


# Reference
- [https://youtu.be/Jh6NthoHYos](https://youtu.be/Jh6NthoHYos){:target="_blank"}
- [https://velog.io/@lena_/Swift-Metatype-ObjectIdentifier](https://velog.io/@lena_/Swift-Metatype-ObjectIdentifier){:target="_blank"}
