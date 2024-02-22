---
title: "[iOS, Swift] Swift Coding Convention"
excerpt: "Swift Coding Convention"
description: "Swift Coding Convention"
modified: 2024-02-22
categories: "iOS"
tags: [iOS, Swift]

toc: true

header:
  teaser: /assets/images/teaser/swift-teaser.png
---


- [https://google.github.io/swift/](https://google.github.io/swift/){: target="_blank"}

# 옵셔널
- as-is

```swift
func test(_ input: String?) -> String {
    return "\(input!) is not null"
}

func test(_ input: String?) -> String {
    if input?.count ?? 0 > 0 {
        return "\(input!) is not null"
    } else {
        return "null"
    }
}
```

- to-be

```swift
func test(_ input: String?) -> String {
    guard let input = input else {
        return "null"
    }

    return "\(input) is not null"
}
```


# 분기처리
- as-is

```swift
func test(_ input: Int) -> String {
    if input < 50 {
        if input < 40 {
            if input < 30 {
                if input < 20 {
                    if input < 10 {
                        return "0~"
                    } else {
                        return "10~"
                    }
                } else {
                    return "20~"
                }
            } else {
                return "30~"
            }
        } else {
            return "40~"
        }
    } else {
        return "50~"
    }
}
```

- to-be

```swift
// guard
func testGuard(_ input: Int) -> String {
    guard input < 50 else {
        return "50~"
    }

    guard input < 40 else {
        return "40~"
    }

    guard input < 30 else {
        return "30~"
    }

    guard input < 20 else {
        return "20~"
    }

    guard input < 10 else {
        return "10~"
    }

    return "0~"
}

// switch
func testSwitch(_ input: Int) -> String {
    switch input {
    case 0..<10:
        return "0~"
    case 10..<20:
        return "10~"
    case 20..<30:
        return "20~"
    case 30..<40:
        return "30~"
    case 40..<50:
        return "40~"
    default:
        return "50~"
    }
}
```

# 루프
- as-is

```swift
for item in collection {
  if item.hasProperty {
    // ...
  }
}
```

- to-be

```swift
for item in collection where item.hasProperty {
  // ...
}
```

# Access Level
- to-be

```swift
extension String {
  public var isUppercase: Bool {
    // ...
  }

  public var isLowercase: Bool {
    // ...
  }
}
```
