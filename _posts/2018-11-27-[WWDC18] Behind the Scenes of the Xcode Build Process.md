---
title: "[WWDC18] Behind the Scenes of the Xcode Build Process"
excerpt: "[WWDC18] Behind the Scenes of the Xcode Build Process"
description: "[WWDC18] Behind the Scenes of the Xcode Build Process"
modified: 2018-11-27
categories: "WWDC18"
tags: [WWDC18, Xcode]

classes: wide
toc: false

header:
  teaser: /assets/images/teaser/wwdc18-teaser.jpg
---

# Behind the Scenes of the Xcode Build Process

## Build System
### Build Process
Build System -> Clang and Swift -> Linker
- 빌드 프로세스는 일련의 테스크를 실행하는 것
### Build Task Execution Order
![스크린샷 2018-11-21 오후 5.06.29.png](/assets/images/post/wwdc18/xcode/스크린샷 2018-11-21 오후 5.06.29.png)
### Build Task Dependency Order
![스크린샷 2018-11-21 오후 5.07.37.png](/assets/images/post/wwdc18/xcode/스크린샷 2018-11-21 오후 5.07.37.png)
- Build tasks are executed in dependency order
### Build Process Graph
![스크린샷 2018-11-21 오후 5.11.45.png](/assets/images/post/wwdc18/xcode/스크린샷 2018-11-21 오후 5.11.45.png)

### Discovered Dependencies
![스크린샷 2018-11-21 오후 5.14.23.png](/assets/images/post/wwdc18/xcode/스크린샷 2018-11-21 오후 5.14.23.png)

### Change Detection and Task Signatures
• 각 테스크는 시그니쳐를 갖고 있다.
• 빌드 시스템은 이전과 현재 빌드의 각 테스크 시그니쳐를 트래킹하고 있다.
• 하나의 테스크 실행을 결정하기위해 시그니쳐들을 비교한다.

### How can you help the build system?
• 각 태스크의 실행 순서보다는 태스크 디펜던시에 대해 생각해봐야 한다.

### 좀 더 빠른 빌드를 위해..
 - Declare Inputs and Outputs

    ![스크린샷 2018-11-26 오후 3.19.08.png](/assets/images/post/wwdc18/xcode/스크린샷 2018-11-26 오후 3.19.08.png)

 - Avoid Auto-Link for Project Dependencies
    ![스크린샷 2018-11-21 오후 5.28.47.png](/assets/images/post/wwdc18/xcode/스크린샷 2018-11-21 오후 5.28.47.png)

 - Add Explicit Dependencies
    ![스크린샷 2018-11-21 오후 5.28.52.png](/assets/images/post/wwdc18/xcode/스크린샷 2018-11-21 오후 5.28.52.png)

## Clang Builds
### What Is Clang?
Apple’s official compiler for the C language family
• C
• C++
• Objective-C
• Objective-C++

### Clang's Two Feature
- Header Maps : Xcode빌드시스템과 clang 컴파일러간의 연결
- Clang Module : Clang speep up(header를 찾는 효율성 증가)


### What Are Header Maps?
![스크린샷 2018-11-21 오후 5.45.43.png](/assets/images/post/wwdc18/xcode/스크린샷 2018-11-21 오후 5.45.43.png)

### Common Project Issues with Header Maps
• 해더는 프로젝트의 일부가 아니다
• 같은 이름의 헤더들을 충돌없이 구분해야 한다

### How do we find system headers?
![스크린샷 2018-11-21 오후 5.48.03.png](/assets/images/post/wwdc18/xcode/스크린샷 2018-11-21 오후 5.48.03.png)


### Clang Modules
• On-disk cached header representation
• Reusable
• Faster build times

### Module map

- Foundation Framework
![스크린샷 2018-11-21 오후 5.57.27.png](/assets/images/post/wwdc18/xcode/스크린샷 2018-11-21 오후 5.57.27.png)
```obj-c
// Module Map - Foundation.framework/Modules/module.modulemap
 framework module Foundation [extern_c] [system] {
 umbrella header "Foundation.h"
 export *
 module * {
 export *
 }
 explicit module NSDebug {
 header "NSDebug.h"
 export *
 }
 }
```
```obj-c
// Foundation.h
...
#import <Foundation/NSScanner.h>
#import <Foundation/NSSet.h>
#import <Foundation/NSSortDescriptor.h>
#import <Foundation/NSStream.h>
#import <Foundation/NSString.h>
#import <Foundation/NSTextCheckingResult.h>
#import <Foundation/NSThread.h>
#import <Foundation/NSTimeZone.h>
...
```

### Module Cache
![스크린샷 2018-11-21 오후 5.58.57.png](/assets/images/post/wwdc18/xcode/스크린샷 2018-11-21 오후 5.58.57.png)



## Swift Builds
### Swift Does Not Have Headers
•  해더가 없기때문에 초보자들이 쉽게 시작할 수 있다
•  분리된 파일들에서 헤더선언이 중복되는 것을 피할 수 있다
•  컴파일러가 선언부분을 찾는것이 용이하다


### Finding Declarations Within a Swift Target

- 컴파일러는 타겟안의 모든 파일을 대상으로 한다.
- Xcode 9: Repeated Work in Debug Builds
    ![스크린샷 2018-11-21 오후 6.18.34.png](/assets/images/post/wwdc18/xcode/스크린샷 2018-11-21 오후 6.18.34.png)
• 각 파일을 분리해서 컴파일
• 페럴러하게 컴파일 하지만, 점점 복잡도가 증가
• 컴파일러는 선언한 부분을 찾기위해 반복적으로 파싱할수 밖에 없다
- Xcode 10: More Sharing in Debug Builds (NEW)
    ![스크린샷 2018-11-21 오후 6.18.40.png](/assets/images/post/wwdc18/xcode/스크린샷 2018-11-21 오후 6.18.40.png)
• 여러 파일을 그룹화하여 공통의 컴파일 프로세스에 넣는다
• 하나의 프로세스안에서는 파싱한 부분을 공유한다
• 오직 프로세스간의 이동에서만 파싱을 반복한다


### Finding Declarations from Objective-C
- 컴파일러는 Clang을 통해 Objective-c를 라이브러리화 한다
    ![스크린샷 2018-11-21 오후 6.19.56.png](/assets/images/post/wwdc18/xcode/스크린샷 2018-11-21 오후 6.19.56.png)
- Objective-C Declarations Come from Headers
    - Imported Objective-C frameworks:
        - Within frameworks that mix Swift and Objective-C
        - Within applications and unit test bundles
- Clang Importer Makes Methods More “Swifty”
![스크린샷 2018-11-21 오후 6.23.21.png](/assets/images/post/wwdc18/xcode/스크린샷 2018-11-21 오후 6.23.21.png)
- Rename Methods Based on Part of Speech
![스크린샷 2018-11-21 오후 6.24.25.png](/assets/images/post/wwdc18/xcode/스크린샷 2018-11-21 오후 6.24.25.png)


### Generating Interfaces to Use in Objective-C
- Compiler Generates Objective-C Header
    ![스크린샷 2018-11-21 오후 6.27.32.png](/assets/images/post/wwdc18/xcode/스크린샷 2018-11-21 오후 6.27.32.png)
- Changing the Class Name in Objective-C
    ![스크린샷 2018-11-21 오후 6.27.38.png](/assets/images/post/wwdc18/xcode/스크린샷 2018-11-21 오후 6.27.38.png)


## Linking

### The Linker
- 마지막 테스크는 실행가능한 목적파일들을 빌드 하는 것
- 모든 컴파일러의 결과물을 하나의 파일로 결합함
- 두가지 종류의 파일들이 사용됨
    • Object files (.o)
    • Libraries (.dylib, .tbd, .a)



# Building Faster in Xcode
- 빌드 효율성 증가
- 리빌드 워크 감소

## Increasing Build Efficiency

### 1. Parallelizing your build process
#### Game Dependency
- List of all targets to build
- Dependency between targets
![스크린샷 2018-11-23 오전 9.50.53.png](/assets/images/post/wwdc18/xcode/스크린샷 2018-11-23 오전 9.50.53.png)

- Serialized Build TimeLine
![스크린샷 2018-11-23 오전 9.51.28.png](/assets/images/post/wwdc18/xcode/스크린샷 2018-11-23 오전 9.51.28.png)

#### Parallelized Build Timeline
![스크린샷 2018-11-23 오전 9.52.20.png](/assets/images/post/wwdc18/xcode/스크린샷 2018-11-23 오전 9.52.20.png)

 - 빌드의 양은 줄지 않는다
 - 빌드하는 시간만 줄어든다
 - 하드웨어의 이용이 증가한다

![스크린샷 2018-11-23 오전 9.53.34.png](/assets/images/post/wwdc18/xcode/스크린샷 2018-11-23 오전 9.53.34.png)

#### Parallelized Target Build Process (NEW)
- OLD : Target Build Process
![스크린샷 2018-11-23 오전 9.55.23.png](/assets/images/post/wwdc18/xcode/스크린샷 2018-11-23 오전 9.55.23.png)
- NEW : Parallelized Target Build Process
![스크린샷 2018-11-23 오전 9.55.27.png](/assets/images/post/wwdc18/xcode/스크린샷 2018-11-23 오전 9.55.27.png)
  - 소스 컴파일이 일찍 시작된다
  - 필요한 부분만 기다린다
  - Run Script phases는 꼭 기다려야 한다

### 2. Measuring your build time
#### Build With Timing Summary (NEW)
![스크린샷 2018-11-23 오전 10.05.29.png](/assets/images/post/wwdc18/xcode/스크린샷 2018-11-23 오전 10.05.29.png)
![스크린샷 2018-11-23 오전 10.05.54.png](/assets/images/post/wwdc18/xcode/스크린샷 2018-11-23 오전 10.05.54.png)

### 3. Dealing with complex expressions
#### a. Use Explicit Types for Complex Properties
- before
```swift
struct ContrivedExample {
    var bigNumber = [4, 3, 2].reduce(1) { soFar, next in
        pow(next, soFar)
    }
}
```
- after (: Double)
```swift
struct ContrivedExample {
    var bigNumber: Double = [4, 3, 2].reduce(1) { soFar, next in
        pow(next, soFar)
    }
}
```


#### b. Provide Types in Complex Closures
- before
```swift
func sumNonOptional(i: Int?, j: Int?, k: Int?) -> Int? {
    return [i, j, k].reduce(0) { soFar, next in
        soFar != nil && next != nil ? soFar! + next! : (soFar != nil ? soFar! : (next != nil ? next! : nil))
    }
}
```
- after
```swift
func sumNonOptional(i: Int?, j: Int?, k: Int?) -> Int? {
    return [i, j, k].reduce(0) { (soFar: Int?, next: Int?) -> Int? in
        soFar != nil && next != nil ? soFar! + next! : (soFar != nil ? soFar! : (next != nil ? next! : nil))
    }
}
```


#### c. Break Apart Complex Expressions
```swift
func sumNonOptional(i: Int?, j: Int?, k: Int?) -> Int? {
  return [i, j, k].reduce(0) { soFar, next in

    if soFar != nil && next != nil {
      return soFar! + next!
    }

    if soFar != nil {
      return soFar!
    }

    if next != nil { return next! }

    return nil
  }
}
```
```swift
func sumNonOptional(i: Int?, j: Int?, k: Int?) -> Int? {
  return [i, j, k].reduce(0) { soFar, next in

    if let soFar = soFar {
      if let next = next {
        return soFar + next
      }

      return soFar
    } else {
      return next
    }
  }
}
```

#### d. Use AnyObject Methods and Properties Sparingly
![스크린샷 2018-11-23 오전 10.16.51.png](/assets/images/post/wwdc18/xcode/스크린샷 2018-11-23 오전 10.16.51.png)

- 프로토콜 정의
![스크린샷 2018-11-23 오전 10.17.21.png](/assets/images/post/wwdc18/xcode/스크린샷 2018-11-23 오전 10.17.21.png)

## Reducing the Work on Rebuilds

### 1. Declaring script inputs and outputs (Run Script Phases)


### 2. Understanding dependencies in Swift
#### a. Incremental Builds Are File-Based
![스크린샷 2018-11-23 오전 10.19.57.png](/assets/images/post/wwdc18/xcode/스크린샷 2018-11-23 오전 10.19.57.png)
![스크린샷 2018-11-23 오전 10.20.08.png](/assets/images/post/wwdc18/xcode/스크린샷 2018-11-23 오전 10.20.08.png)
![스크린샷 2018-11-23 오전 10.20.15.png](/assets/images/post/wwdc18/xcode/스크린샷 2018-11-23 오전 10.20.15.png)
![스크린샷 2018-11-23 오전 10.20.22.png](/assets/images/post/wwdc18/xcode/스크린샷 2018-11-23 오전 10.20.22.png)

#### b. Dependencies Within a Target Are Per-File
![스크린샷 2018-11-23 오전 10.22.06.png](/assets/images/post/wwdc18/xcode/스크린샷 2018-11-23 오전 10.22.06.png)

#### c. Cross-Target Dependencies Are Coarse-Grained
![스크린샷 2018-11-23 오전 10.22.14.png](/assets/images/post/wwdc18/xcode/스크린샷 2018-11-23 오전 10.22.14.png)

#### d. Swift Dependency Rules
- 컴파일러는 보수적
- 함수안에 내용이 바뀌어도 파일 인터페이스에는 영향을 안줌
- 모듈안의 디펜던시는 파일 단위
- 디펜던시간의 across 타겟은 전체 타겟

### 3. Limiting your Objective-C/Swift interface
#### a. Mixed-Source App Targets
![스크린샷 2018-11-23 오전 10.36.18.png](/assets/images/post/wwdc18/xcode/스크린샷 2018-11-23 오전 10.36.18.png)

#### b. Keep Your Generated Header Minimal (swift)
- Use private when possible
    ![스크린샷 2018-11-26 오후 3.57.08.png](/assets/images/post/wwdc18/xcode/스크린샷 2018-11-26 오후 3.57.08.png)
    ![스크린샷 2018-11-26 오후 3.57.01.png](/assets/images/post/wwdc18/xcode/스크린샷 2018-11-26 오후 3.57.01.png)
- Use block-based APIs
- Migrate to Swift 4
- Turn off “Swift 3 @objc Inference”

#### c. Keep Your Bridging Header Minimal (objective-c)
- Use categories to break up your interface
- before
![스크린샷 2018-11-23 오전 10.40.37.png](/assets/images/post/wwdc18/xcode/스크린샷 2018-11-23 오전 10.40.37.png)
- after
![스크린샷 2018-11-23 오전 10.40.46.png](/assets/images/post/wwdc18/xcode/스크린샷 2018-11-23 오전 10.40.46.png)

#### d. Less Content, Fewer Changes, Faster Builds
- 더 적은 컨텐츠, 더 적은 수정이 빌드를 빠르게 한다.

# Reference
- [https://developer.apple.com/videos/play/wwdc2018/415/](https://developer.apple.com/videos/play/wwdc2018/415/){:target="_blank"}
