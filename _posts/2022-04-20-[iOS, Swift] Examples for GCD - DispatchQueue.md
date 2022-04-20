---
title: "[iOS, Swift] Examples for GCD - DispatchQueue"
excerpt: "how to use a dispatchqueue"
description: "how to use a dispatchqueue"
modified: 2022-04-20
categories: "iOS"
tags: [iOS, Swift, GCD, DispatchQueue, Serial, Concurrent, Sync / Async]

toc: true

header:
  teaser: /assets/images/teaser/swift-teaser.png
---

# Serial vs Concurrent
## Serial Queue
- A serial queue requires one task to be completed before proceeding with the next task

```swift
public func executeInSerialQueue() {
    let serialQueue = DispatchQueue(label: "serialQueue")

    serialQueue.async {
        for i in 0..<3 {
            print("🟥 \(i) in \(#function)")
        }
    }

    serialQueue.async {
        for i in 0..<3 {
            print("🟦 \(i) in \(#function)")
        }
    }
}

//🟥 0 in executeInSerialQueue()
//🟥 1 in executeInSerialQueue()
//🟥 2 in executeInSerialQueue()
//🟦 0 in executeInSerialQueue()
//🟦 1 in executeInSerialQueue()
//🟦 2 in executeInSerialQueue()
```

## Concurrent Queue
- Concurrent queue starts the next task even if one task is not completed

```swift
public func executeInConcurrentQueue() {
    let concurrentQueue = DispatchQueue(label: "concurrentQueue", attributes: .concurrent)

    concurrentQueue.async {
        for i in 0..<3 {
            print("🟥 \(i) in \(#function)")
        }
    }

    concurrentQueue.async {
        for i in 0..<3 {
            print("🟦 \(i) in \(#function)")
        }
    }
}

//🟦 0 in executeInConcurrentQueue()
//🟥 0 in executeInConcurrentQueue()
//🟦 1 in executeInConcurrentQueue()
//🟥 1 in executeInConcurrentQueue()
//🟥 2 in executeInConcurrentQueue()
//🟦 2 in executeInConcurrentQueue()
```

# Sync vs Async
- Synchronously starting a task will block the calling thread until the task is finished
- Asynchronously starting a task will directly return on the calling thread without blocking

## Sync in a SerialQueue
```swift
public func executeSyncInSerialQueue() {
    let serialQueue = DispatchQueue(label: "serialQueue")

    print("Start")
    serialQueue.sync {
        for i in 0..<3 {
            print("🟥 \(i) in \(#function)")
        }
    }

    serialQueue.sync {
        for i in 0..<3 {
            print("🟦 \(i) in \(#function)")
        }
    }
    print("End")
}

//Start
//🟥 0 in executeSyncInSerialQueue()
//🟥 1 in executeSyncInSerialQueue()
//🟥 2 in executeSyncInSerialQueue()
//🟦 0 in executeSyncInSerialQueue()
//🟦 1 in executeSyncInSerialQueue()
//🟦 2 in executeSyncInSerialQueue()
//End
```

## Sync in a ConcurrentQueue
```swift
public func executeSyncInConcurrentQueue() {
    let concurrentQueue = DispatchQueue(label: "concurrentQueue", attributes: .concurrent)

    print("Start")
    concurrentQueue.sync {
        for i in 0..<3 {
            print("🟥 \(i) in \(#function)")
        }
    }

    concurrentQueue.sync {
        for i in 0..<3 {
            print("🟦 \(i) in \(#function)")
        }
    }
    print("End")
}

//Start
//🟥 0 in executeSyncInConcurrentQueue()
//🟥 1 in executeSyncInConcurrentQueue()
//🟥 2 in executeSyncInConcurrentQueue()
//🟦 0 in executeSyncInConcurrentQueue()
//🟦 1 in executeSyncInConcurrentQueue()
//🟦 2 in executeSyncInConcurrentQueue()
//End
```

## Async in a SerialQueue
- Since it's a serial queue, the red task that came in first completes and then the blue task starts

```swift
public func executeAsyncInSerialQueue() {
    let serialQueue = DispatchQueue(label: "serialQueue")

    print("Start")
    serialQueue.async {
        for i in 0..<3 {
            print("🟥 \(i) in \(#function)")
        }
    }

    serialQueue.async {
        for i in 0..<3 {
            print("🟦 \(i) in \(#function)")
        }
    }
    print("End")
}

//Start
//🟥 0 in executeAsyncInSerialQueue()
//End
//🟥 1 in executeAsyncInSerialQueue()
//🟥 2 in executeAsyncInSerialQueue()
//🟦 0 in executeAsyncInSerialQueue()
//🟦 1 in executeAsyncInSerialQueue()
//🟦 2 in executeAsyncInSerialQueue()
```

## Async in a ConcurrentQueue
```swift
public func executeAsyncInConcurrentQueue() {
    let concurrentQueue = DispatchQueue(label: "concurrentQueue", attributes: .concurrent)

    print("Start")
    concurrentQueue.async {
        for i in 0..<3 {
            print("🟥 \(i) in \(#function)")
        }
    }

    concurrentQueue.async {
        for i in 0..<3 {
            print("🟦 \(i) in \(#function)")
        }
    }
    print("End")
}

//Start
//End
//🟥 0 in executeAsyncInConcurrentQueue()
//🟦 0 in executeAsyncInConcurrentQueue()
//🟥 1 in executeAsyncInConcurrentQueue()
//🟦 1 in executeAsyncInConcurrentQueue()
//🟥 2 in executeAsyncInConcurrentQueue()
//🟦 2 in executeAsyncInConcurrentQueue()
```

# Reference
- [https://www.avanderlee.com/swift/concurrent-serial-dispatchqueue/](https://www.avanderlee.com/swift/concurrent-serial-dispatchqueue/){:target="_blank"}
- [https://zeddios.tistory.com/516](https://zeddios.tistory.com/516){:target="_blank"}
