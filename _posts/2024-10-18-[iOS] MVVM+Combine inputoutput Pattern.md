---
title: "[iOS] MVVM+Combine input/output Pattern"
excerpt: "MVVM+Combine input/output Pattern"
description: "MVVM+Combine input/output Pattern"
modified: 2024-10-18
categories: "iOS"
tags: [iOS, MVVM, Combine]

toc: true

header:
  teaser: /assets/images/teaser/ios-teaser.png
---

# a ViewModel to use `@Published` of combine for binding data
```swift 
class ObservableViewModel: ObservableObject {
    @Published private(set) var models: [Model]?
    private var bag: Set<AnyCancellable> = Set<AnyCancellable>()
    
    func fetch() {
        DataSource
            .shared
            .get()
            .sink { completion in
                if case .failure(let error) = completion {
                    print("Error : \(error)")
                }
            } receiveValue: { models in
                print("Results : \(models)")
                self.models = models
            }
            .store(in: &bag)
    }
}
```

# input/output Pattern
## ViewModel
```swift 
protocol ViewModelType {
    associatedtype InputType
    associatedtype OutputType
    
    func trans(_ input: AnyPublisher<InputType, Never>) -> AnyPublisher<OutputType, Error>
}

enum Input {
    case fetch
}

enum Output {
    case loadData([Model])
}

class ViewModel: ViewModelType {
    private var bag: Set<AnyCancellable> = Set<AnyCancellable>()
    
    func trans(_ input: AnyPublisher<Input, Never>) -> AnyPublisher<Output, Error> {
        return input
            .flatMap({ input -> AnyPublisher<Output, Error> in
                switch input {
                case .fetch:
                    return DataSource
                        .shared
                        .get()
                        .map { models in
                            return .loadData(models)
                        }
                        .eraseToAnyPublisher()
                }
                
            })
            .eraseToAnyPublisher()
    }
}
```

## Model & DataSource
```swift 
struct Model {
    let title: String
}

final class DataSource {
    static let shared = DataSource()
    private init() {}
    
    func get() -> AnyPublisher<[Model], Error> {
        return Just(["title1", "title2"])
            .setFailureType(to: Error.self)
            .map({ $0.map({ Model(title: $0) }) })
            .eraseToAnyPublisher()
    }
}
```

## View
```swift
class View: UIView {
    let viewModel: ViewModel = ViewModel()
    private var bag: Set<AnyCancellable> = Set<AnyCancellable>()
    private let input: PassthroughSubject<ViewModel.InputType, Never> = .init()
    
    override init(frame: CGRect) {
        super.init(frame: frame)
        
        bind()
        input.send(.fetch)
    }

    required init?(coder: NSCoder) {
        fatalError("init(coder:) has not been implemented")
    }
    
    func bind() {
        viewModel
            .trans(input.eraseToAnyPublisher())
            .sink { completion in
                if case .failure(let error) = completion {
                    print("Error : \(error)")
                }
            } receiveValue: { output in
                if case .loadData(let models) = output {
                    print("Results : \(models)")
                }
            }
            .store(in: &bag)
    }
}
```

# Reference
- [https://velog.io/@jakkujakku98/Combine-INPUTOUTPUT-패턴](https://velog.io/@jakkujakku98/Combine-INPUTOUTPUT-패턴)