---
title: "[iOS, Swift] Clean Architecture With MVVM on iOS(using SwiftUI, Combine, SPM)"
excerpt: "Clean Architecture, MVVM DesignPattern"
description: "Clean Architecture, MVVM DesignPattern"
modified: 2021-01-18
categories: "iOS"
tags: [iOS, Swift, SwiftUI, Combine, SPM, Clean Architecture, MVVM DesignPattern, Architecture, Design Pattern]

toc_sticky: false

header:
  teaser: /assets/images/teaser/swift-teaser.png
---

# 개요

- [Clean Architecture](/etc/The-Clean-Architecture/){: target="_blank"}
- [MVVM (View->ViewModel->Model)](/etc/MVVM-Pattern/){: target="_blank"}
- [SwiftUI](https://developer.apple.com/kr/xcode/swiftui/){: target="_blank"}
- [Combine](https://developer.apple.com/documentation/combine){: target="_blank"}
- [SwiftPackageManager](https://swift.org/package-manager/){: target="_blank"}

여러 iOS 프로젝트에 사용하는 Architecture들 중, Clean Architecture와 MVVM 패턴을 사용하여 프로젝트 구조를 설계해 봅니다.

Clean Architecture와 MVVM 패턴을 프로젝트에 어떻게 적용할지 살펴보고, 이를 이용하여 `일별 날씨정보를 보여주는 앱`을 예제로 만들어 봅니다.

먼저 프로젝트의 구조는 `Clean Architecture`를 이용하여 역할별 레이어들로 분리합니다.

이렇게 분리된 레이어들은 타겟으로 만들어 `Swift Package Manager`에 적용하면 레이어별 의존성 관리를 쉽게 할 수 있습니다.

UI부분은 `SwiftUI`와 `MVVM 패턴`으로 적용하여 쉽게 유지보수 할 수 있도록 합니다.

특히, MVVM 패턴에서 View와 ViewModel사이에는 `Combine`을 사용하여 Data Binding을 간단히 처리합니다.

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
  - `Presentation Layer`: UI관련 레이어
  - `Domain Layer`: 비지니스 룰과 로직 담당 레이어
  - `Data Layer`: 원격/로컬등 외부에서 데이터를 가져오는 레이어

![cleanArchitecture](/assets/images/post/cleanArchitecture/cleanarchitecture.png)

- 각 레이어들의 `Dependency` 방향은 모두 원밖에서 원안쪽으로 향하고 있음
- 레이어중 UI부분을 담당하는 `Presentation Layer` 부분은 아래 이미지와 같은 `MVVM 패턴`으로 적용

![mvvm](/assets/images/post/mvvm/MVVMPattern.png)

## 0-2. Clean Architecture 구조내의 데이터 흐름 및 의존성 방향
![dataFlow](/assets/images/post/cleanArchitecture/dataFlow.png)
- 여기서 주목해야 될 점은, DomainLayer와 DataLayer 사이에서는 Dependency Inversion으로 구현된 부분
- Dependency Inversion 구현을 통해, 원내부에서 원밖으로 실행을 시킬수 있는 구조가 가능, 이는 ViewModel(PresentationLayer)에서 UseCase(DomainLayer)를 통해 Repository(DataLayer)의 데이터를 받아서 사용할 수 있는 구조로 개발이 가능하게함

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


        //MARK: - Tests
        .testTarget(
            name: "DataLayerTests",
            dependencies: ["DataLayer"]),

        .testTarget(
            name: "DomainLayerTests",
            dependencies: ["DomainLayer"]),

        .testTarget(
            name: "PresentationLayerTests",
            dependencies: ["PresentationLayer"]),
    ]
)
```
- Clean Architecture의 각 Layer별 의존성 구현

# 2. Domain Layer 구현
- Business Logic 구현 Layer

## 2-1. Entity
### WeatherEntity.swift
```swift
import Foundation

public struct WeatherEntity: Identifiable {
    public let id: String
    public let icon: String
    public let location: String
    public let temperature: Float
    public let description: String
    public let date: Date

    public init(id: String, icon: String, location: String, temperature: Float, description: String, date: Date)
    {
        self.id = id
        self.icon = icon
        self.location = location
        self.temperature = temperature
        self.description = description
        self.date = date
    }
}
```
- 원의 가장 내부 계층
- `외부 변화에 변경될 가능성이 없고`, Business Rule의 핵심기능을 담당하는 데이터구조
- 상위계층에 의존성을 갖고 있지 않아 독립적으로 비지니스 기능을 수행할 수 있어야 함

## 2-2. UseCase
### FetchDailyWeatherUseCase.swift
```swift
import Foundation
import Combine

public protocol FetchDailyWeatherUseCaseInterface {
    func execute(completion: @escaping (Result<[WeatherEntity], Error>) -> Void) -> Cancellable?
}

public final class FetchDailyWeatherUseCase: FetchDailyWeatherUseCaseInterface {

    private let repository: WeatherRepositoryInterface

    public init(repository: WeatherRepositoryInterface) {
        self.repository = repository
    }

    public func execute(completion: @escaping (Result<[WeatherEntity], Error>) -> Void) -> Cancellable? {
        return repository.fetchDailyWeather { result in
            completion(result)
        }
    }
}

public protocol WeatherRepositoryInterface {
    func fetchDailyWeather(completion: @escaping (Result<[WeatherEntity], Error>) -> Void) -> Cancellable?
}
```
- `Business Logic`을 처리하는 부분
- `execute()` 부분이 Business Logic을 처리하는 부분, 별도의 비지니스 로직이 필요하면 이곳에 추가
- `Dependency Inversion`
  - DataLayer에서 구현될 WeatherRepository에 대한 인터페이스(`WeatherRepositoryInterface`)를 DomainLayer에서 선언을 함으로써, DomainLayer와 DataLayer 간의 `Dependency Inversion` 구현을 가능하게 함
  - 즉, 하위 계층인 DomainLayer에서 상위 계층의 DataLayer의 호출 부분을 알 수 있게 됨

# 3. Presentation Layer 구현
- UI 구현 Layer
- `MVVM 패턴`으로 구현
- View와 ViewModel 사이는 `Combine`으로 `DataBinding` 처리

## 3-1. ViewModel
### DailyWeatherViewModel.swift
```swift
import Foundation
import Combine
import DomainLayer

public protocol DailyWeatherViewModelInput {
    func executeFetch()
}

public protocol DailyWeatherViewModelOutput {
    var dailyWeather: [WeatherEntity] { get }
}

public final class DailyWeatherViewModel: ObservableObject, DailyWeatherViewModelInput, DailyWeatherViewModelOutput {

    private let useCase: FetchDailyWeatherUseCaseInterface

    @Published public var dailyWeather: [WeatherEntity] = []

    public init(useCase: FetchDailyWeatherUseCaseInterface) {
        self.useCase = useCase
    }

    public func executeFetch() {
        var _ = useCase.execute { result in
            switch result {
            case .success(let dailyWeather):
                self.dailyWeather = dailyWeather
            case .failure:
                self.dailyWeather = []
            }
        }
    }
}
```
- `ObservableObject`와 `@Published` 등의 Combine을 이용한 바인딩 처리
- ViewModel이 Model(Entity)에 대해 의존성을 갖음(ViewModel -> Model)

## 3-2. View
### DailyWeatherView.swift
```swift
import SwiftUI

public struct DailyWeatherView: View {

    @ObservedObject public var viewModel: DailyWeatherViewModel

    public init(viewModel: DailyWeatherViewModel) {
        self.viewModel = viewModel
    }

    public var body: some View {
        ScrollView() {
            Text("Daily Weather")
                .font(.title)
                .fontWeight(.bold)

            Spacer(minLength: 20)

            VStack(spacing: 40) {
                ForEach(self.viewModel.dailyWeather) { weather in
                    WeatherView(icon: weather.icon, location: weather.location, temperature: weather.temperature, date: weather.date)
                }
            }
        }
        .padding(EdgeInsets(top: 20, leading: 0, bottom: 0, trailing: 0))
        .onAppear {
            self.viewModel.executeFetch()
        }
    }

}

private struct WeatherView: View {

    let icon: String
    let location: String
    let temperature: Float
    let date: Date

    private var formatter: DateFormatter = {
        let formatter = DateFormatter()
        formatter.dateFormat = "yyyy. MM. dd."
        return formatter
    }()

    init(icon: String, location: String, temperature: Float, date: Date) {
        self.icon = icon
        self.location = location
        self.temperature = temperature
        self.date = date
    }

    var body: some View {
        HStack(spacing: 20) {

            Image(icon, bundle: Bundle.module)

            VStack(alignment: .leading) {
                Text(formatter.string(from: date))
                    .font(.body)
                    .foregroundColor(.gray)
                Text(location)
                    .font(.title)

                Spacer()

                Text(String(format: " %.1f °C", temperature))
                    .font(.title)
                    .fontWeight(.bold)
            }
        }
        .padding(10)
        .background(Color.gray.opacity(0.2))
        .cornerRadius(25)
    }
}
```
- `@ObservedObject`로 ViewModel안에서 업데이트되는 Model값 사용
- View가 ViewModel에 대해 의존성을 갖음(View -> ViewModel)

# 4. Data Layer 구현
- DB 및 외부 API등을 통해 내부/외부 데이터를 사용하는 Layer

## 4-1. Repository
### WeatherRepository.swift
```swift
import Foundation
import Combine
import DomainLayer

public final class WeatherRepository: WeatherRepositoryInterface {

    private let dataSource: WeatherDataSourceInterface

    public init(dataSource: WeatherDataSourceInterface) {
        self.dataSource = dataSource
    }

    public func fetchDailyWeather(completion: @escaping (Result<[WeatherEntity], Error>) -> Void) -> Cancellable? {

        return dataSource.fetchDailyWeather { result in
            switch result {
            case .success(let dailyWeather):
                var weatherEntities = [WeatherEntity]()
                for weather in dailyWeather {
                    weatherEntities.append(weather.dto())
                }
                completion(.success(weatherEntities))
            case .failure(let error):
                completion(.failure(error))
            }
        }
    }
}
```
- DataSource를 통해 데이터 값을 가져오고, 해당 모델을 Domain Layer 에서 사용할 수 있는 Entity등의 포맷으로 전환 시켜주는 부분

## 4-2. DataSource
- DB 및 외부 API등을 통해 데이터를 가져오는 부분
- API Framework(Alamofire)등으로 사용할 수 있음

### WeatherDTO.swift (DataModel & DTO(Data Transfer Object))
```swift
import Foundation
import DomainLayer

public struct WeatherDTO: Codable {
    let weather: WeatherDataDTO
    let main: WeatherMainDTO
    let name: String
    let dt: TimeInterval

    // DTO: Data Transfer Object
    public func dto() -> WeatherEntity {
        return WeatherEntity(id: UUID().uuidString, icon: weather.icon, location: name, temperature: Float(main.temp), description: weather.description, date: Date(timeIntervalSince1970: dt))
    }
}

public struct WeatherDataDTO: Codable {
    let main: String
    let description: String
    let icon: String
}

public struct WeatherMainDTO: Codable {
    let temp: Double
    let temp_min: Double
    let temp_max: Double
}
```
- `DataSource`에서 데이터를 파싱하는 모델
- 파싱된 데이터 값은 `DTO(Data Transfer Object)`를 통해 Domain Layer에서 사용하는 `Entity`로 변환
- 데이터 파싱부분을 `DataModelDTO로 만들어서 사용`하기 때문에, DataSource(외부 api, DB등)의 속성 변경에 대해 `DTO부분만 수정`되고, Entity에는 전혀 영향이 없음(Entity는 외부 요인에 의해 변경이 없어야함)
- DataLayer에서의 변경사항이 다른 계층에 영향을 주지 않음

### WeatherDataSource.swift (WeatherLocalDataSource)
```swift
import Foundation
import Combine
import DomainLayer

public protocol WeatherDataSourceInterface {
    func fetchDailyWeather(completion: @escaping (Result<[WeatherDTO], Error>) -> Void) -> Cancellable?
}

public final class WeatherLocalDataSource: WeatherDataSourceInterface {

    public init() {}

    public func fetchDailyWeather(completion: @escaping (Result<[WeatherDTO], Error>) -> Void) -> Cancellable? {
        return Just(dailyWeatherLocalData)
            .tryMap { try JSONSerialization.data(withJSONObject: $0, options: .prettyPrinted) }
            .decode(type: [WeatherDTO].self, decoder: JSONDecoder())
            .replaceError(with: [])
            .eraseToAnyPublisher()
            .sink { dailyWeather in
                completion(.success(dailyWeather))
            }
    }
}
```
- 데이터들을 파싱하는 부분
- DB 및 API Network Framework 등을 이용하여, 내부/외부 데이터 값을 받아오도록 구현 가능

# 5. App Layer (의존성 주입 컨테이너)
- 앱의 진입점
- 의존성 주입 컨테이너 및 환경 설정
- `Clean Architecture` 기반 구조에 의존성을 주입하기 위해 컨테이너 형태로 구현

![dependencyDiagram](/assets/images/post/cleanArchitecture/dependencyDiagram.png)

## 5-1. DI(Dependency Injection)
### AppDI.swift
- 앱 환경에 따른 의존성 주입 부분
- AppDI는 모든 DI를 사용하는 컨테이너 역할 및 앱 환경설정 구현

```swift
import Foundation
import DataLayer
import DomainLayer
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

    public func dailyWeatherDependencies() -> DailyWeatherViewModel {

        //MARK: Data Layer
        let dataSource: WeatherDataSourceInterface

        switch appEnvironment.phase {
        case .DEV:
            dataSource = WeatherLocalDataSource()
        default:
            dataSource = WeatherLocalDataSource()
        }

        let repository = WeatherRepository(dataSource: dataSource)

        //MARK: Domain Layer
        let useCase = FetchDailyWeatherUseCase(repository: repository)

        //MARK: Presentation
        let viewModel = DailyWeatherViewModel(useCase: useCase)

        return viewModel
    }
}
```

### AppDIInterface.swift
- `AppDIInterface`는 [PresentationLayer](#3-presentation-layer-구현)안에 구현 (SubView들에 대한 의존성 주입을 상위 View에서 해줘야 하는 경우가 생기기 때문에, View와 동일한 레이어에 인터페이스 존재)
- `AppLayer`는 최상위 레이어, `AppLayer`->`PresentationLayer` 하위 레이어들에 대해 의존성을 갖음

```swift
public protocol AppDIInterface {
    func dailyWeatherDependencies() -> DailyWeatherViewModel
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
            DailyWeatherView(viewModel: AppDI.shared.dailyWeatherDependencies())
        }
    }
}
```

# Example App (DailyWeatherApp)
![DailyWeatherApp](/assets/images/post/cleanArchitecture/example.png)

# SourceCode
- [https://github.com/tigi44/CleanArchitectureWithMVVM](https://github.com/tigi44/CleanArchitectureWithMVVM){: target="_blank"}

# Reference
- [https://tech.olx.com/clean-architecture-and-mvvm-on-ios-c9d167d9f5b3](https://tech.olx.com/clean-architecture-and-mvvm-on-ios-c9d167d9f5b3){: target="_blank"}
- [https://medium.com/swift2go/clean-architecture-for-massivetobe-mobile-apps-bf8e44a98b37](https://medium.com/swift2go/clean-architecture-for-massivetobe-mobile-apps-bf8e44a98b37){: target="_blank"}
- [https://medium.com/@jang.wangsu/di-dependency-injection-이란-1b12fdefec4f](https://medium.com/@jang.wangsu/di-dependency-injection-이란-1b12fdefec4f){: target="_blank"}
