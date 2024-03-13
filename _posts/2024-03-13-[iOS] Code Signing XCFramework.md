---
title: "[iOS] Code Signing XCFramework"
excerpt: "Code Signing XCFramework"
description: "Code Signing XCFramework"
modified: 2024-03-13
categories: "iOS"
tags: [iOS, CodeSigning, XCFramework]

classes: wide
toc: false

header:
  teaser: /assets/images/teaser/ios-teaser.png
---

# 이슈
- 코드 서명 ID를 사용하여 XCFramework에 서명하면 프레임워크를 사용하는 개발자에게 프레임워크를 누가 개발했으며, 서명을 추가한 이후로 변조되지 않았음을 증명합니다.

# Sign the XCFramework bundle

```
% codesign --timestamp -s <identity> xcframeworks/MyLibrary.xcframework
% codesign --timestamp -s "Apple Distribution: ..." xcframeworks_PATH/MyLibrary.xcframework
```

- Apple Developer Program의 회원으로서 배포 프레임워크에 서명하려면, 코드 서명 ID는 Apple Distribution 또는 Apple Development ID여야 합니다. 엔터프라이즈 프로그램의 구성원으로서 배포를 위한 프레임워크에 서명하려면, iOS 배포 또는 iOS 앱 개발 ID를 사용하세요.
- `-s` 옵션 뒤에 코드 서명 ID의 전체 이름을 제공할 필요가 없습니다. XCFramework에 서명하는 데 사용하는 키체인에서 코드 서명 ID를 고유하게 식별하는 문자열을 사용하세요.
- `--timestamp` 옵션은 서명시 secure timestamp 값을 넣어주며, 해당 시점이후에 xcframework가 변조되었는지를 판단하기 위해 사용됩니다.

# Reference
- [https://developer.apple.com/documentation/xcode/creating-a-multi-platform-binary-framework-bundle/](https://developer.apple.com/documentation/xcode/creating-a-multi-platform-binary-framework-bundle/){: target="_blank"}
- [https://developer.apple.com/documentation/xcode/verifying-the-origin-of-your-xcframeworks#Diagnose-build-failures-caused-by-code-signature-changes](https://developer.apple.com/documentation/xcode/verifying-the-origin-of-your-xcframeworks#Diagnose-build-failures-caused-by-code-signature-changes){: target="_blank"}
