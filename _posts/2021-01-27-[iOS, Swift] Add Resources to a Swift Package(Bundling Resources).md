---
title: "[iOS, Swift] Add Resources to a Swift Package (Bundling Resources)"
excerpt: "Add Resources to a Swift Package (Bundling Resources)"
description: "Add Resources to a Swift Package (Bundling Resources)"
modified: 2021-01-27
categories: "iOS"
tags: [iOS, Swift, SwiftUI, SPM, Resources, Bundle]

toc: true

header:
  teaser: /assets/images/swift-teaser.png
---

Swift Package Manager(SPM)에 리소스 포함이 가능하다.
예시로 SPM에 `Color Assets`와 `json으로 된 File`을 리소스로 추가하여 사용해보고, package 외부에서 번들을 통해 해당 리소스에 접근하는 방법들을 살펴보자.

## 0. Xcode detects a resource automatically
아래 항목들은 Apple의 기본 리소스 타입들로, 별도의 설정없이도 XCode가 자동으로 리소스 인식을 해준다.
- Interface Builder files; for example, XIB files and storyboards
- Core Data files; for example, xcdatamodeld files
- Asset catalogs
- .lproj folders you use to provide localized resources

## 1. Add Resources to a Swift Package

- 먼저 아래와 같이 SPM 타겟 소스안에 `Resources/` 경로로 리스소들을 추가해준다.
  - Resources/ColorAssets.xcassets
  - Resources/jsonFile.json
- ColorAssets에는 CustomColor 값을 하나 추가한다.

![ColorAssets](/assets/images/post/ios/spm/spm_resources.png)

## 2. Setup Package.swift for using resources
- `ColorAssets.xcassets`의 경우 기본 리소스 타입으로 자동 인식되므로, 별도의 설정없이 사용 가능하다.
- `jsonFile.json`은 `ResourcesSPM` 타겟에 `resources: [.process("Resources/jsonFile.json")]`로 추가한다.

```swift
import PackageDescription

let package = Package(
    name: "ResourcesSPM",
    platforms: [.iOS(.v14)],
    products: [
        // Products define the executables and libraries a package produces, and make them visible to other packages.
        .library(
            name: "ResourcesSPM",
            targets: ["ResourcesSPM"]),
    ],
    dependencies: [
        // Dependencies declare other packages that this package depends on.
        // .package(url: /* package url */, from: "1.0.0"),
    ],
    targets: [
        // Targets are the basic building blocks of a package. A target can define a module or a test suite.
        // Targets can depend on other targets in this package, and on products in packages this package depends on.
        .target(
            name: "ResourcesSPM",
            dependencies: [],
            resources: [.process("Resources/jsonFile.json")]),

        .testTarget(
            name: "ResourcesSPMTests",
            dependencies: ["ResourcesSPM"]),
    ]
)
```

## 3. Using Resources in code
- 위에서 셋팅한 리소스들을 사용할 소스코드를 만들어준다.
- 여기서 주의할 부분은 해당 Package내의 번들에서 리소스를 가져와야 하기 때문에, `Bundle.module`를 이용하여 리소스들을 가져와야 한다는 것이다.

```swift
// ColorAssets.xcassets
public extension UIColor {
    @available(iOS 11.0, *)
    static let customColor: UIColor? = UIColor(named: "CustomColor", in: Bundle.module, compatibleWith: nil)
}

// jsonFile.json
public class JsonFile {
    static func read() -> Any? {
        if let path = Bundle.module.path(forResource: "jsonFile", ofType: "json") {
            do {
                let data = try Data(contentsOf: URL(fileURLWithPath: path), options: [])
                let jsonResult = try JSONSerialization.jsonObject(with: data, options: [])
                return jsonResult
              } catch {
                return nil
              }
        } else {
            return nil
        }
    }
}
```

## 4. Make tests
- 이제 테스트케이스를 통해, 리소스들을 잘 가져오는지 확인해 보자.
- 리소스를 정상적으로 가져왔다면, `UIColor.customColor`와 `JsonFile.read()`가 nil이 아닌 값으로 할당되어야 한다.

```swift
import XCTest
@testable import ResourcesSPM

final class ResourcesSPMTests: XCTestCase {
    func testColorAssets() {
        XCTAssertNotNil(UIColor.customColor)
        XCTAssertNil(UIColor(named: "noCustomColor", in: Bundle.main, compatibleWith: nil))
    }

    func testJsonFile() {
        let jsonFile = JsonFile.read()
        XCTAssertNotNil(jsonFile)
    }

    static var allTests = [
        ("testColorAssets", testColorAssets),
        ("testJsonFile", testJsonFile),
    ]
}
```

## 5. Access a Resource from External
- Swift Package 안에 있는 리소스를 Package 외부 혹은 다른 Target에서 접근하여 사용하는 방법을 알아보자.
- 외부에서 리소스를 사용하기 위해서는, 리소스가 있는 Package내의 리소스 번들 경로를 외부에 공개해야 한다.

### Share Resources
- 기본적으로 해당 Package의 bundle을 외부에 공개하여, 외부에서 리소스에 접근하도록 할 수 있다.
- 각 리소스들은 bundle의 path or url 을 통해 리소스에 직접 접근하도록 할 수 있는데, `jsonFile.json`는 path로 외부에 공개하여, 리소스에 직접 접근할 수 있도록 하였다.

```swift
import Foundation

public enum SharedResource {
    static public let bundle: Bundle = Bundle.module
    static public let jsonPath: String? = Bundle.module.path(forResource: "jsonFile", ofType: "json")
}

```

### External the package
- 공유된 bundle을 통해, 해당 package내의 리소스에 접근한다.
- package 내부에서 리소스를 사용할때와 차이점은, `Bundle.module` 대신 공유된 `SharedResource.bundle`을 사용하여 리소스를 사용한다는 것이다.

```swift
import Foundation
import UIKit
import ResourcesSPM

public extension UIColor {
    @available(iOS 11.0, *)
    static let externalCustomColor: UIColor? = UIColor(named: "CustomColor", in: SharedResource.bundle, compatibleWith: nil)
}

public class ExternalJsonFile {
    static func read() -> Any? {
        if let path = SharedResource.jsonPath {
            do {
                let data = try Data(contentsOf: URL(fileURLWithPath: path), options: [])
                let jsonResult = try JSONSerialization.jsonObject(with: data, options: [])
                return jsonResult
              } catch {
                return nil
              }
        } else {
            return nil
        }
    }
}
```

### Tests
- 공유된 리소스들을 사용하는 부분을 테스트케이스를 통해 확인한다.

```swift
final class ExternalTests: XCTestCase {
    func testExternalColorAssets() {
        XCTAssertNotNil(UIColor.externalCustomColor)
    }

    func testExternalJsonFile() {
        let jsonFile = ExternalJsonFile.read()
        XCTAssertNotNil(jsonFile)
    }

    static var allTests = [
        ("testExternalColorAssets", testExternalColorAssets),
        ("testExternalJsonFile", testExternalJsonFile),
    ]
}
```

## SourceCode
- [ResourcesSPM](https://github.com/tigi44/ResourcesSPM){: target="_blank"}

## Reference
- [https://developer.apple.com/documentation/swift_packages/bundling_resources_with_a_swift_package](https://developer.apple.com/documentation/swift_packages/bundling_resources_with_a_swift_package){: target="_blank"}
