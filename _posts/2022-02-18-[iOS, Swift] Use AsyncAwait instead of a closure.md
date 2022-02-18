---
title: "[iOS, Swift] Use AsyncAwait instead of a Closure"
excerpt: "Use AsyncAwait instead of a Closure"
description: "Use AsyncAwait instead of a Closure"
modified: 2022-02-18
categories: "iOS"
tags: [iOS, Swift, SwiftUI, Async/Await, closure]

toc: true

header:
  teaser: /assets/images/teaser/swift-teaser.png
---

# Use a closure
- use a result type for a Error
- escaping completion closure

```swift
func fetch(completion: @escaping (Result<FetchedData, NSError>) -> Void) {
    post(url: "https://api-url",
         param: nil) { result in

           completion(result)
    }
}
```

# Async/Await
## withCheckedContinuation
```swift
func fetch() async -> Result<FetchedData, NSError> {
    return await withCheckedContinuation({ continuation in
      post(url: "https://api-url",
           param: nil) { result in

            continuation.resume(returning: result)
        }
    })
}
```

## withCheckedThrowingContinuation
```swift
func fetch() async throws -> FetchedData {
    return try await withCheckedThrowingContinuation({ continuation in
      post(url: "https://api-url",
           param: nil) { result in

            switch result {
            case .success(let data):
                continuation.resume(returning: data)
            case .failure(let error):
                continuation.resume(throwing: error)
            }
        }
    })
}
```

## Async Map

```swift
extension Sequence {
    public func asyncMap<T>(
        _ transform: (Element) async throws -> T
    ) async rethrows -> [T] {
        var values = [T]()

        for element in self {
            try await values.append(transform(element))
        }

        return values
    }
}

...

func fetch(dataList: [FetchedData]) async -> [FetchedData] {
    return await dataList.asyncMap({ await $0.asyncFunction() })
}
```
