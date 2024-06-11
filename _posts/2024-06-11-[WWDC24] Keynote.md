---
title: "[WWDC24] Keynote"
excerpt: "WWDC24 Keynote"
description: "WWDC24 Keynote"
modified: 2024-06-11
categories: "WWDC24"
tags: [WWDC, WWDC24, Keynote, AppleIntelligence]

toc: true

header:
  teaser: /assets/images/teaser/wwdc24-teaser.png
---


# iOS 18
- https://developer.apple.com/wwdc24/101?time=869
## Home Screen Layout

![Inline-image-1.png](/assets/images/post/wwdc24/Inline-image-1.png)

- new look for App Icons
- light, dark, tint 모드에 따라 icon이 다르게 랜더링 됨

## Controls API

![Inline-image-2.png](/assets/images/post/wwdc24/Inline-image-2.png)

- 위젯처럼, 앱내 기능을 제어센터에 표시
- on Lock Screen 화면 및 action button등에서 활용 가능

### Control 정의

- control gallery에 생성됨

```swift
struct CarClimateControl: ControlWidget ‹
    var body: some ControlWidgetConfiguration {
        StaticControlConfiguration(kind: "car.climate") {
        ControlWidgetToggle( "Climate", isOn: isCooling, action: ToggleClimated)) {
            Label($0 ? "Cooling" : "Climate Off", image: "car.fan")
                .symbolEffect(rotate.byLayer)
            }
        ｝
        .displayName ("Climate")
    ｝
}
```

# iPadOS 18
- https://developer.apple.com/wwdc24/101?time=2584

## TabBar

# watchOS 11
- https://developer.apple.com/wwdc24/101?time=2156

## Smart Stack
- iOS의 LiveActivities 기능 확장
- Widget group
    - iOS의 상호작용 위젯 활용
- RelavantContext

## Double Tap
- handGestureShortcut 기능을 이용해 더블탭 동작 수행 기능 구현

```swift
var body: some View {
    Button(action: action) {
        Label(playPayse, systemImage: "playhouse.fill")
    }
    .handGestureShortcut(.primaryAction)
}
```

# macOS Sequoia
- https://developer.apple.com/wwdc24/101?time=3052
## Continuity
### iPhone mirroring

![Inline-image-3.png](/assets/images/post/wwdc24/Inline-image-3.png)

- Mac 에서 내 아이폰 화면 실행 가능
- 앱 실행 및 드래그앤드랍으로 파일 공유등 가능

# visionOS 2
- https://developer.apple.com/wwdc24/101?time=342

## TabletopKit
![Inline-image-4.png](/assets/images/post/wwdc24/Inline-image-4.png)

# Apple Intelligence (Siri)
- https://developer.apple.com/wwdc24/101?time=3870
## App Intents API
- SiriKit
## App Entities
- Spotlight API
- 이를 통해 색인을 등록할 수 있음
## Image Playground API
### Genmoji
- AttributedString 으로 생성
    - 기존 emoji는 Text
- SwiftUI - Genmoji View

```swift
.buttonStyle(.profileButtonStyle)
.imagePlaygroundSheet (
    isPresented: $isImagePlaygroundPresented,
    concept: "Cat Watching Movies"
) { createdImageURL in
    do {
        // A place to store the new profile image.
        let finalURL = URL.documentsDirectory.appendingPathComponent ("profilePicture_" +
        UUID().uuidString, conformingTo: fileType(createdImageURL))

        try FileManager.default moveItem(at: createdImageURL, to: FinalURL)
        createdAvatar = finalURL
    ｝ catch｛
        // Handle profile picture save error.
    }
}
```

# Xcode 16
- https://developer.apple.com/wwdc24/102?time=1270
## Predictive Code Completion
- Swift & Apple SDK 전용으로 훈련됨

## Swift Assist (24년 하반기 출시)
- https://developer.apple.com/wwdc24/102?time=1438
- 코딩관련 질문, 새 API 실험등의 태스크 수행
- 기호와 기호간의 관계를 포함하여 프로젝트내 세부사항을을 사용해 맞춤형 코드 생성
    - 프로젝트내의 리소스를 활용하여 코드 생성 가능
- 프레임워크를 최신화하여 새로운 프레임워크로 적용이 가능

# Swift
- C++ -> Swift
- 다양한 OS에서 지원
    - visual studio code 지원
- http://www.GitHub.com/swiftlang

## Swift 6
- 동시성 개선
- Data-race safety
    - 컴파일시 동작, 위험성 제거
- http://swiftpackageindex.com

## Swift Testing
- 새로운 테스트 프레임워크

```swift
@Test ("Continents mentioned in videos",
        .tags(.location),
        arguments: [ ("A Beach", [Continent.oceanial]),
        ("By the Lake", [.europel]),
        ("Camping in the Woods", [.northAmerica, .asial]),
        ("Ocean Breeze", [.oceanial]),
        ("Patagonia Lake", [.southAmerical]),
        ("China Paddy Field", [.asial]),
        ("Scotland Coast", [.europel]),
        ("Grand Canyon Evening", [.northAmerical]),
        ("Liwa Horizon", [.asial]),
      ])
func continent (videoName: String, expectedContinents: [Continent]) async throws {
    let videoLibrary = try await VideoLibrary()
    let video = try #require (await videoLibrary.video (named: videoName) )
    #expect(video.mentionedContinents == expectedContinents)
｝
```

- `@Test("")` : 테스트 제목 추가
    - `.tags`: 태깅을 통해 테스트 선택 가능
- `#expect` : 결과 평가

## SwiftUI
- 다양한 플랫폼으로 공유 가능
- 모든 UI 프레임워크와 공통 foundation을 공용함
    - gesture 인식 부분이 UIKit에서 제거 되면서, 쉽게 SwiftUI에서 활용 가능
    - uiview animate도 SwiftUI에서 활용 가능

## SwiftData
- index와 unique 기능 추가
