---
title: "[iOS, Swift] Clean Architecture With MVVM on iOS(using SwiftUI, Combine, SPM)"
excerpt: "Clean Architecture, MVVM DesignPattern"
description: "Clean Architecture, MVVM DesignPattern"
modified: 2021-01-18
categories: "iOS"
tags: [iOS, Swift, SwiftUI, Combine, SPM, Clean Architecture, MVVM DesignPattern, Architecture, Design Pattern]

toc_sticky: false

header:
  teaser: /assets/images/swift-logo.png
---

<!-- # Clean Architecture With MVVM on iOS(using SwiftUI, Combine, SwiftPackageManager) -->
- [Clean Architecture](/etc/The-Clean-Architecture/){: target="_blank"}
- [MVVM (View->ViewModel->Model)](/etc/MVVM-Pattern/){: target="_blank"}
- [SwiftUI](https://developer.apple.com/kr/xcode/swiftui/){: target="_blank"}
- [Combine](https://developer.apple.com/documentation/combine){: target="_blank"}
- [SwiftPackageManager](https://swift.org/package-manager/){: target="_blank"}

여러 iOS 프로젝트에 사용하는 Architecture들 중, Clean Architecture와 MVVM 패턴을 사용하여 프로젝트 구조를 설계해 봅니다.

`Clean Architecture` 구조를 이용하여 앱을 역할별 레이어들로 분리합니다.

이렇게 분리된 레이어들은 타겟으로 만들어 `Swift Package Manager`에 적용하면 레이어별 의존성 관리를 쉽게 할 수 있습니다.

전체적인 앱 동작은 `Switf의 Combine`을 기반으로 하여 비동기적인 이벤트 기반의 동작을 할 수 있도록 구현합니다.

UI부분은 `SwiftUI`와 `MVVM 패턴`으로 적용하여 쉽게 유지보수 할 수 있도록 합니다.

iOS 프로젝트에 Clean Architecture를 적용하기전에 먼저 `의존성 주입`에 관해 알아야 전체적인 구조 이해에 도움이 됩니다.

## 의존성 주입 (DI: Dependency Injection)
### 의존성 (Dependency)
```swift
class A {
  var number: Int = 1
}

class B {
  var a: A
  init() {
    a = A()
  }
  func printNumber() {
    print(a.number)
  }
}

let b: B = B()
b.printNumber()
```
- B클래스는 A클래스에 의존하고 있음
- A클래스의 수정이 B클래스에 영향을 주고, B클래스를 재활용하기에 용이하지 않음

### 주입 (Injection)
```swift
class A {
  var number: Int = 1
}

class B {
  var a: A
  init(a: A) {
    self.a = a
  }
  func printNumber() {
    print(a.number)
  }
}

let b: B = B(a: A())
b.printNumber()
```
- B클래스가 의존하고 있던 A클래스를 B클래스 밖에서 주입시킴
- B클래스와 A클래스간의 의존성을 줄이기위한 시도이지만, 아직 의존성이 완전히 분리되진 않음

### 의존성 분리 및 의존관계 역전 (IOC: Inversion Of Control)
```swift
protocol AInterface {
  var number: Int { get set }
}
class A: AInterface {
  var number: Int = 1
}
class C: AInterface {
  var number: Int = 2
}
class D: AInterface {
  var number: Int = 3
}

class B {
  var a: AInterface
  init(a: AInterface) {
    self.a = a
  }
  func printNumber() {
    print(a.number)
  }
}

let b: B = B(a: A())
b.printNumber()

let b: B = B(a: C())
b.printNumber()

let b: B = B(a: D())
b.printNumber()
```
- A클래스의 추상화 작업을 통해 B클래스와 A클래스간의 의존성을 분리시킴
- 기존 A클래스이외에 AInterface로 구현될 어떠한 클래스들(C클래스, D클래스)이 모두 외부에서 B클래스에 주입될 수 있음. 이것을 의존관계 역전이라고 함. (B클래스가 의존성을 가지는 클래스를 B클래스 밖에서 주입시킬 수 있음)


# 0. Clean Architecture With MVVM 구조

## 0-1. Clean Architecture 와 MVVM 구조 사용
- 전체적인 앱 구조는 Clean Architecture 기반으로 레이어링
  - `Presentation Layer`: UI관련 레이어(View, ViewModel)
  - `Domain Layer`: 비지니스 로직 담당(Use Case)
  - `Data Layer`: 원격/로컬 데이터소스에서 데이터를 가져옴

![cleanArchitecture](/assets/images/post/cleanArchitecture/cleanarchitecture.png)

- 각 레이어들의 Dependency 방향은 아래와 같이 원안쪽으로 향하고 있음

![withMVVM](/assets/images/post/cleanArchitecture/withMVVM.png)

- 레이어중 UI부분을 담당하는 Presentation Layer 부분은 MVVM 디자인 패턴으로 적용
  - `MVVM 패턴`: VIEW -Dependency-> VIEWMODEL -Dependency-> MODEL

## 0-2. Clean Architecture 구조내의 데이터 흐름 및 의존성 방향
![dataFlow](/assets/images/post/cleanArchitecture/dataFlow.png)
- 여기서 주목해야 될 점은, DomainLayer와 DataLayer 사이에서는 Dependency Inversion으로 구현된 부분
- Dependency Inversion 구현을 통해, 원내부에서 원밖으로 실행을 시킬수 있는 구조가 가능, 이는 ViewModel(PresentationLayer)에서 UseCase(DomainLayer)를 통해 Repository(DataLayer)의 데이터를 받아서 사용할 수 있는 구조로 개발이 가능해짐

# 1. SPM(Swift Package Manager)를 이용한 앱 구조 및 Layer별 의존성 구현
## 1-1. 프로젝트 구조
![spm](/assets/images/post/cleanArchitecture/spm.png)

## 1-2. Package.swift

```swift
import PackageDescription

let package = Package(
    name: "CleanArchitectureWithMVVMSPM",
    platforms: [.iOS(.v14), .macOS("10.15")],
    products: [
        .library(
            name: "CleanArchitectureWithMVVMSPM",
            targets: ["DataLayer", "DomainLayer", "PresentationLayer"]),

    ],
    dependencies: [
    ],
    targets: [

        //MARK: - Data Layer
        // Dependency Inversion : UseCase(DomainLayer) <- Repository <-> DataSource
        .target(
            name: "DataLayer",
            dependencies: ["DomainLayer"]),

        //MARK: - Domain Layer
        .target(
            name: "DomainLayer",
            dependencies: []),

        //MARK: - Presentation Layer (MVVM)
        // Dependency : View -> ViewModel -> Model(DomainLayer)
        .target(
            name: "PresentationLayer",
            dependencies: ["DomainLayer"]),
    ]
)
```
- Clean Architecture의 각 Layer별 의존성 구현

# 2. Domain Layer 구현
## 2-1. Entity
### GroupEntity.swift
```swift
import Foundation

public struct MyGroupEntity: Identifiable {
    public let id: String
    public let image: String
    public let name: String
    public let date: String

    public init(id: String, image: String, name: String, date: String) {
        self.id = id
        self.image = image
        self.name = name
        self.date = date
    }
}
```
- 외부 변화에 변경될 가능성이 가장 적은 데이터구조

## 2-2. UseCase
### FetchMyGroupListUseCase.swift
```swift
import Foundation
import Combine

public protocol FetchMyGroupListUseCaseInterface {
    func execute(completion: @escaping (Result<[MyGroupEntity], Error>) -> Void) -> Cancellable?
}

public final class FetchMyGroupListUseCase: FetchMyGroupListUseCaseInterface {

    private let groupRepository: GroupRepositoryInterface

    public init(groupRepository: GroupRepositoryInterface) {
        self.groupRepository = groupRepository
    }

    public func execute(completion: @escaping (Result<[MyGroupEntity], Error>) -> Void) -> Cancellable? {
        return groupRepository.fetchMyGroupList { result in
            completion(result)
        }
    }
}

public protocol GroupRepositoryInterface {
    func fetchMyGroupList(completion: @escaping (Result<[MyGroupEntity], Error>) -> Void) -> Cancellable?
}
```
- `execute()` 부분이 Business Logic을 처리하는 부분, 별도의 비지니스 로직이 필요하면 이곳에 추가
- `GroupRepositoryInterface` 부분과 처음부분에서 설명한 [의존성 주입 (DI: Dependency Injection)](#의존성-주입-di-dependency-injection) 부분이 DomainLayer와 DataLayer 간의 `Dependency Inversion` 구현을 가능하게 함

# 3. Presentation Layer 구현
- MVVM 디자인 패턴으로 구현
- VIEW -Dependency-> VIEWMODEL -Dependency-> MODEL(Entity)

## 3-1. ViewModel
### MyGroupListViewModel.swift
```swift
import Foundation
import Combine
import DomainLayer

public protocol MyGroupListViewModelInput {
    func executeFetch()
}

public protocol MyGroupListViewModelOutput {
    var myGroups: [MyGroupEntity] { get }
}

public final class MyGroupListViewModel: ObservableObject, MyGroupListViewModelInput, MyGroupListViewModelOutput {

    private let fetchMyGroupListUseCase: FetchMyGroupListUseCaseInterface

    @Published public var myGroups: [MyGroupEntity] = []

    public init(fetchMyGroupListUseCase: FetchMyGroupListUseCaseInterface) {
        self.fetchMyGroupListUseCase = fetchMyGroupListUseCase
    }

    public func executeFetch() {
        var _ = fetchMyGroupListUseCase.execute { result in
            switch result {
            case .success(let myGroups):
                self.myGroups = myGroups
            case .failure:
                self.myGroups = []
            }
        }
    }
}
```
- `ObservableObject`와 `@Published` 등의 Combine을 이용한 바인딩 처리
- ViewModel이 Model(Entity)에 대해 의존성을 갖음(ViewModel -> Model)

## 3-2. View
### MyGroupListView.swift
```swift
import SwiftUI

public struct GroupView: View {

    struct MyGroupView: View {

        let image: String
        let name: String
        let date: String

        var body: some View {
            VStack(alignment: .leading, spacing: 10) {
                Image(image)
                    .resizable()
                    .frame(width: 130, height: 130, alignment: .center)
                    .cornerRadius(5)
                VStack(alignment: .leading) {
                    Text(name)
                        .font(.headline)
                        .fontWeight(.regular)
                    Text(date)
                        .font(.footnote)
                }
            }
        }
    }

    @ObservedObject public var viewModel: MyGroupListViewModel

    public init(viewModel: MyGroupListViewModel) {
        self.viewModel = viewModel
    }

    public var body: some View {
        VStack(alignment: .leading, spacing: 20) {
            HStack {
                Text("내 그룹 목록")
                    .font(.title)
                    .fontWeight(.bold)
                Spacer()
            }

            ScrollView(.horizontal, showsIndicators: false) {
                HStack(spacing: 10) {
                    ForEach(self.viewModel.myGroups) { myGroupEntity in
                        MyGroupView(image: myGroupEntity.image, name: myGroupEntity.name, date: myGroupEntity.date)
                    }
                }
            }
        }
        .onAppear {
            self.viewModel.executeFetch()
        }
    }
}
```
- `@ObservedObject`로 ViewModel안에서 업데이트되는 Model값 사용
- View가 ViewModel에 대해 의존성을 갖음(View -> ViewModel)

# 4. Data Layer 구현
## 4-1. Repository
### GroupRepository.swift
```swift
import Foundation
import Combine
import DomainLayer

public protocol GroupRepositoryInterface {
    func fetchMyGroupList(completion: @escaping (Result<[MyGroupEntity], Error>) -> Void) -> Cancellable?
}

public final class GroupRepository: GroupRepositoryInterface {

    private let groupDataSource: GroupDataSourceInterface

    public init(groupDataSource: GroupDataSourceInterface) {
        self.groupDataSource = groupDataSource
    }

    public func fetchMyGroupList(completion: @escaping (Result<[MyGroupEntity], Error>) -> Void) -> Cancellable? {

        return groupDataSource.fetchMyGroupList { result in
          switch result {
          case .success(let myGroupModels):
              var myGroupEntities = [MyGroupEntity]()
              for myGroupModel in myGroupModels {
                  myGroupEntities.append(myGroupModel.dtoMyGroupEntity())
              }
              completion(.success(myGroupEntities))
          case .failure(let error):
              completion(.failure(error))
          }
        }
    }
}
```
- `DataSource`를 통해 데이터 값을 가져오고, 해당 모델을 Domain Layer 에서 사용할 수 있는 Entity등의 포맷으로 전환 시켜주는 부분

## 4-2. DataSource
- DB 및 외부 API등을 통해 데이터를 가져오는 부분

### DataModel & DTO(Data Transfer Object) (GroupDataSource.swift)
```swift
public struct GroupModelDTO: Codable {
    let image: String
    let name: String
    let date: String

    // DTO: Data Transfer Object
    public func dtoMyGroupEntity() -> MyGroupEntity {
        return MyGroupEntity(id: self.name, image: self.image, name: self.name, date: self.date)
    }
}
```
- `DataSource`에서 가져오는 데이터 모델
- 데이터 값을 [Domain Layer](#2-domain-layer-구현)에서 사용하는 Entity값으로 변환가능(Data Transfer Object)

### GroupLocalDataSource (GroupDataSource.swift)
```swift
import Foundation
import Combine
import DomainLayer

public protocol GroupDataSourceInterface {
    func fetchMyGroupList(completion: @escaping (Result<[GroupModelDTO], Error>) -> Void) -> Cancellable?
}

public final class GroupLocalDataSource: GroupDataSourceInterface {
    static var myGroups = [
        [
            "image": "group1",
            "name": "group 1",
            "date": "2020.01.01 Mon",
        ],
        [
            "image": "group2",
            "name": "group 2",
            "date": "2020.01.02 Mon",
        ]
    ]

    public init() {}

    public func fetchMyGroupList(completion: @escaping (Result<[GroupModelDTO], Error>) -> Void) -> Cancellable? {
        return Just(GroupLocalDataSource.myGroups)
            .tryMap { try JSONSerialization.data(withJSONObject: $0, options: .prettyPrinted) }
            .decode(type: [GroupModelDTO].self, decoder: JSONDecoder())
            .replaceError(with: [])
            .eraseToAnyPublisher()
            .sink { myGroups in
                completion(.success(myGroups))
            }
    }
}

```
- `DataSource`의 한 종류로 테스트를 위해 앱 내부의 값을 사용 하도록 구현
- Combine의 Just(Publisher)로 구현

### GroupRemoteDataSource (GroupDataSource.swift)
```swift
public protocol GroupRemoteDataSourceInterface {
    init(urlString: String)
}

public final class GroupRemoteDataSource: GroupDataSourceInterface, GroupRemoteDataSourceInterface {

    let urlString: String

    public init(urlString: String) {
        self.urlString = urlString
    }

    public func fetchMyGroupList(completion: @escaping (Result<[GroupModelDTO], Error>) -> Void) -> Cancellable? {

        return URLSession
            .shared
            .dataTaskPublisher(for: URL(string: urlString)!)
            .map(\.data)
            .decode(type: [GroupModelDTO].self, decoder: JSONDecoder())
            .replaceError(with: [])
            .eraseToAnyPublisher()
            .sink { myGroups in
                completion(.success(myGroups))
            }
    }
}
```
- 외부 API 호출할 수 있는 `DataSource`
- URLSession을 Combine구조로  이용할 수 있는 dataTaskPublisher 사용

# 5. App Layer (의존성 주입 컨테이너)
- 앱의 진입점
- 의존성 주입 컨테이너 및 환경 설정
- `Clean Architecture` 기반 구조에 의존성을 주입하기 위해 컨테이너 형태로 구현

![dependencyDiagram](/assets/images/post/cleanArchitecture/dependencyDiagram.png)

## 5-1. DI(Dependency Injection)
### MyGroupDI.swift
- 의존성 주입

```swift
import Foundation
import DataLayer
import DomainLayer
import PresentationLayer

public class MyGroupDI {

    private let appEnvironment: AppEnvironment

    public init(appEnvironment: AppEnvironment) {
        self.appEnvironment = appEnvironment
    }

    public func myGroupListDependencies() -> MyGroupListViewModel {

        //MARK: Data Layer
        let groupDS: GroupDataSourceInterface

        switch appEnvironment.phase {
        case .DEV:
            groupDS = GroupLocalDataSource()
        default:
            groupDS = GroupRemoteDataSource(urlString: "")
        }

        let groupRepo = GroupRepository(groupDataSource: groupDS)

        //MARK: Domain Layer
        let fetchMyGroupListUseCase = FetchMyGroupListUseCase(groupRepository: groupRepo)

        //MARK: Presentation
        let myGroupListViewModel = MyGroupListViewModel(fetchMyGroupListUseCase: fetchMyGroupListUseCase)

        return myGroupListViewModel
    }
}
```

### AppDI.swift
- AppDI는 모든 DI를 사용하는 컨테이너 역할 및 앱 환경설정 구현

```swift
import Foundation
import PresentationLayer

enum PHASE {
    case DEV, ALPHA, REAL
}

public class AppEnvironment {
    let phase: PHASE = .DEV
}

public class AppDI: AppDIInterface {

    static let shared = AppDI(appEnvironment: AppEnvironment())

    private let appEnvironment: AppEnvironment

    private init(appEnvironment: AppEnvironment) {
        self.appEnvironment = appEnvironment
    }

    public func myGroupListDependencies() -> MyGroupListViewModel {

        let myGroupDI: MyGroupDI = MyGroupDI(appEnvironment: appEnvironment)

        let myGroupListViewModel = myGroupDI.myGroupListDependencies()

        return myGroupListViewModel
    }
}
```

### AppDIInterface.swift
- `AppDIInterface`는 [PresentationLayer](#3-presentation-layer-구현)안에 구현
- `AppLayer`는 최상위 레이어, `AppLayer`->`PresentationLayer` 하위 레이어들에 대해 의존성을 갖음

```swift
import Foundation

public protocol AppDIInterface {
    func myGroupListDependencies() -> MyGroupListViewModel
}
```

## 5-2. App Main
### CleanArchitectureWithMVVMApp.swift
- 뷰 초기화시에 의존성 주입 컨터이너인 AppDI를 사용하여, 해당 뷰에 맞는 의존성 주입

```swift
import SwiftUI
import PresentationLayer

@main
struct CleanArchitectureWithMVVMApp: App {
    var body: some Scene {
        WindowGroup {
            MyGroupListView(viewModel: AppDI.shared.myGroupListDependencies())
        }
    }
}
```

# SourceCode
- [https://github.com/tigi44/CleanArchitectureWithMVVM](https://github.com/tigi44/CleanArchitectureWithMVVM){: target="_blank"}

# Reference
- [https://tech.olx.com/clean-architecture-and-mvvm-on-ios-c9d167d9f5b3](https://tech.olx.com/clean-architecture-and-mvvm-on-ios-c9d167d9f5b3){: target="_blank"}
- [https://medium.com/swift2go/clean-architecture-for-massivetobe-mobile-apps-bf8e44a98b37](https://medium.com/swift2go/clean-architecture-for-massivetobe-mobile-apps-bf8e44a98b37){: target="_blank"}
- [https://medium.com/@jang.wangsu/di-dependency-injection-이란-1b12fdefec4f](https://medium.com/@jang.wangsu/di-dependency-injection-이란-1b12fdefec4f){: target="_blank"}
