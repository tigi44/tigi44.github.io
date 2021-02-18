---
title: "[iOS, Objective-c] Effective Objective-c 2.0 #1~#4"
excerpt: "Effective Objective-c 2.0 #1~#4"
description: "Effective Objective-c 2.0 #1~#4"
last_modified_at: 2018-01-25T14:00:00+09:00
categories: "iOS"
tags: [iOS, Objective-c, Effective Objective-c]

toc_sticky: false

header:
  teaser: /assets/images/ios-teaser.png
---

# 1. Accustoming Yourself to Objective-c
## 1. Familiaize Yourself with Objective-C's Roots
- Objective-C 는 동적인 언어로 다른 언어들이 컴파일 시간에 하는 것을 objective-C 는 런타임이 한다.
- function calling 이 아닌 messaging 구조를 사용한다.

```
// Messaging
Object *obj = [Object new];
[obj performWith:parameter1 and:parameter2];

// Function calling
Object *obj = new Object;
obj->perform(prarmeter1, parameter2);
```

- 함수 호출은 컴파일러가 어떤 코드를 실행할지 결정하지만, 메세지 전달 방식은 런타임때 어떤 코드를 실행할지 결정한다.
- 메세지를 받을 객체의 타입에 대해 컴파일러가 관리하지 않는다. 이것은 런타임떄 관리되며 동적 바인딩이라는 프로세스를 통해 진행된다. (반환되는 객체에 대한 타입 역시 런타임때 결정)
- 런타임은 개발자의 코드와 해당 코드와 연결된 동적 라이브러리를 잘 연결하는 일을 하며, 런타임이 업데이트되면 성능 향상에 많은 도움이 된다. (컴파일에서 많은 것을 하는 언어는 성능에 도움을 받기위해 컴파일을 다시 해야한다.)
-  Objective-C는 C의 확장언어 이기에 C의 기능을 사용할 수 있으며, 메모리 관리 방식등을 c와 연관지어 생각해볼 수 있다.
-  Objective-C의 객체는 스택에 할당되는 것을 허용하지 않고, 항상 힙에 할당된다.
-  Objective-C 런타임이 retain count 를 통해 메모리 관리를 하기 때문에 개발자가 직접 malloc, free등으로 메모리 관리를 할 필요는 없다.
-  Objective-C 객체는 힙을 사용하기에 메모리 관리 등의 영행으로 성능에 영향을 줄 수있는데, C로 된 구조체등을 사용함으로써 성능상 이점을 얻을 수도 있다.

## 2. Minimize Importing Headers in Headers
- 사용할 클래스의 존재만을 알려주는 경우 `forward declaring the class`라는 방식으로 `@class` 로 선언해주면 된다.
- #include 의 사슬이 길어지면 컴파일시간이 늘어날 수 있다.
- #include 대신 #import를 사용하면 서로 참조하는데 무한 루프에서 벗어날 순 있지만, 제대로 컴파일 되지 않는 클래스가 나올 수 있다.
- 헤더파일에서는 포워드 클래스 선언으로 하고, 구현파일에서 연관된 헤더를 #include 하는 방식이 클래스끼지 연관 되는 것을 최대한 피할 수 있게 해준다.
- 프로토콜을 사용할때는 클래스 확장 카테고리를 사용하는게 낫다.

## 3. Prefer Literal Syntax over the Equivalent Methods
```
// 리터럴 미사용
NSNumber *number = [NSNumber numberWith:1];

// 리터럴 사용
NSNumber *number = @1;

//experssion
int x = 5;
float y = 6.32f;
NSNumber *expressionNumber = @(x + y);

// 리터럴 배열
NSArray *animals = @[@"cat", @"dog", @"mouse", @"badger"];
NSString *dog = animals[1]

// 리터럴 사전
NSDictionary *personData = @{@"firstName" : @"Matt", @"lastName" : @"Galloway", @"age" : @28};
NSString *lastName = personData[@"lastName"];
```
- 리터럴 문법에서는 생성하는 객체중 하나라도 nil이면 exception이 발생한다.

## 4. Prefer Typed Constants to Preprocessor #define
- #define은 전처리기 지시어(preprocessor)로 타입에 대한 정보가 없으며, 해당 #define이 선언된 헤더파일을 포함하는 모든 곳에서 정의된 값이 상수값으로 치환된다.
- 컴파일러를 사용하여 올바른 상수값이 적용되는지 확인하는것이 더 좋은데, 이는 상수를 정의하여 사용하면 된다.
```
// #define
#define ANIMATION_DURATION 0.3
// 상수
static const NSTimeInterval kAnimationDuration = 0.3;
```
- Objective-C는 네임스페이스가 별도로 없기 때문에 헤더 파일등에 선언되면 global 변수로 정의되게 된다. 외부로 노출할 필요가 없는 상수는 구현 파일에 정의하는 것이 좋다.
- static, const 둘 다 사용하여 선언하는 것이 좋다. const는 변수 값을 변경하지 못하도록 해주고, static은 컴파일러가 만드는 object file을 만들어 내는 translation unit 단위(클래스당 한개의 단위 생성)의 지역 변수라는 것을 의미하게 된다. static을 사용하지 않으면 외부 심벌로 받아들여 다른 번역 단위 이름의 변수와 duplaicate symbol 에러가 발생 할 수 있다.
- 상수를 외부로 공개할때는 `extern`을 사용하여 정의하면 된다.

## 5. Use Enumerations for States, Options, and Status Codes
- enum은 에러 상태 코드나 조합할 수 있는 옵션에 사용되는 상수를 정의하는데 많은 도움이 된다.
- 메소드의 옵션등으로 사용할때는 enum값을 비트 쉬프트(<<, 2의 제곱..)방식을 이용하여 OR, AND 연산등이 가능하게 할 수 있다.
- 명시적 타임으로 enum을 선언하기 위해 NS_ENUM, NS_OPTIONS 매크로를 사용 할 수 있다.
- 스위치문에서 enum 사용시 default 문을 구현하지 않으면, 컴파일러에서 enum의 모든 값을 다루는지 체크할 수 있기에 유용하게 사용할 수 있다.

# 2. Objects, Messaging, and the Runtime
## 6. Understand Properties

### autosynthesis
- 프로퍼티로 선언되면 컴파일러는 autosynthesis를 통해 자동으로 접근자 메소드 코드를 생성한다. 코드가 만들어질때 프로퍼티 이름prefix로 `_`를 붙인 인스턴스 변수가 자동 추가된다,
- @synthesis 문법을 통해 인스턴스 변수의 이름을 변경할 수도 있다.
- @dynamic 문법을 통해 접근자 메소드가 자동 생성되는 것을 막을 수도 있다.

### Atomicity Attribute
- 자동 생성되는 접근자 메소드는 기존적으로 lock이 사용되도록 설정된다.
- 이를 막기위해 nonatomic 속성을 줄 수 있다.

### Read/Write Attribute
- readwrite : getter, setter 모두 제공한다.
- readonly : getter만 사용된다.

### Memory-Management Semantics Attributes
- setter에서 새로운 값을 리테인할지, 할당만할지 등을 결정하게 된다.
- assing
- strong
- weak
- unsafe_unretained
- copy

### Method Names Attribute
- getter=<name> : 게터이름정의
- setter=<name> : 세터 이름정의

- @property 문법은 객체의 캡슐화를 정의하는 방법을 제공한다.
- 속성을 사용하여 데이터를 저장하는 올바른 semantics를 제공한다.
- iOS에서 nonatomic 대신 atomic을 사용하여 여러가지 성능 문제가 발생 할 수있다.

## 7. Access Instance Variables Directly When Accessing Them Internally
- 직접 접근과 프로퍼티를 이용하여 접근의 차이
    -  직접 접근이 빠르다. (objective-c method dispatch를 통하지 않음, 컴파일러가 직접 메모리 접근하는 코드 생성)
    -  직접 접근은 setter의 메모리 관리 시멘틱스를 바이패스함.(copy등으로 선언된 프로퍼티의 동작 무시, 프로퍼티에 copy로 선언했으면 초기화에서도 setter를 사용하지 않고 직접 copy로 set해주는게 좋다.)
    -  직접 접근시 kvo 알림이 발생하지 않음.
    -  프로퍼티를 이용하면 getter/setter에서 브레이크 포인트로 디버깅하기 쉬움

- 직접접근과 프로퍼티를 이용하는 방법의 장점을 합치면 get은 직접 접근, set은 프로퍼티 접근. 하지만 이때 주의해야될 점이 있다.
    - 초기화 메소드(initializers) 안에서는 set도 직접 접근하는 방법 사용.(하위 클래스에서 setter를 재정의override할수 있기때문)
    - dealloc안에서도 역시 직접접근
    - 프로퍼티가 지연초기화(lazy initialized)를 사용하면 getter사용(직접 접근하면 초기화될 타이밍을 놓침)

## 8. Understand Object Equality

- `==` 연산자는 포인터 값을 비교한다. 두 객체가 같은지를 비교하려면 `isEqual:`과 같은 메소드를 사용해야 한다.
- 동등비교 메소드

```
- (BOOL)isEqual:(id)object;
- (NSUInteger)hash;
```

- hash 생성

```
- (NSUinteger)hash {
     NSUInteger firstNameHash = [_firstName hash];
     NSUInteger lastNameHash = [_lastName hash];
     NSUInteger ageHash = _age;
     return firstNameHash ^ lastNameHash ^ ageHash;
}
```

- 컨테이너안 가변 클래스의 동등성
    * 객체가 컬랙션에 추가된면 해쉬값을 바꿀 수 없다
    * 컬랙션에 추가된 객체를 수정하면 문제점이 발생할 수 있다. 그에 대해 인지하고 있어야 한다.

- 같은 객체는 항상 해쉬 값이 같아야함. but 해쉬값이 같다고 객체가 같을 필욘 없다.
-  동등성을 테스트할때는 모든 프로퍼티를 비교하는것 보단 필요한 프로퍼티만 비교.
-  해쉬 메소드를 만들때 빨라야 하지만, 충돌도 최소화해야 한다.

## 9. Use the Class Cluster Pattern to Hide Implementation Detail
- class cluster 는 abstract base class 뒤에 상세 구현을 숨길 수 있는 좋은 구현 방법이다.
- Objective-C 시스템 전반에 걸쳐 많이 사용된다.

```
// abstract base class
typedef NS_ENUM(NSUInteger, EOCEmployeeType){
     EOCEmployeeTypeDeveloper,
     EOCEmployeeTypeDesigner,
     EOCEmployeeTypeFinance,
}

@interface EOCEmployee : NSObject

@property (copy) NSString *name;
@property NSUInteger salary;

+ (EOCEmployee*) employeeWithType:(EOCEmployeeType)type;

- (void)doADaysWork;

@end

@implementation EOCEmployee

+ (EOCEmployee*)employeeWithType:(EOCEmployeeType)type{
     switch(type){
          case EOCEmployeeTypeDeveloper;
               return [EOCEmployeeDeveloper new];
               break;
          case EOCEmployeeTypeDesigner:
               return [EOCEmployeeDesigner new];
               break;
          case EOCEmployeeTypeFinance:
               return [EOCEmployeeFinance new];
               break;
     }
}

- (void)doADaysWork{
     // Subclasses implement this.
}

@end

// concrete subclass
@interface EOCEmployeeDeveloper : EOCEmployee
@end

@implementation EOCEmployeeDeveloper
- (void)doADaysdWork{
     [self writeCode];
}
@end
```

### Class Clusters in Cocoa
- 시스템 프레인워크에 많은 class cluster가 있다. NSArray, NSMutableArray와 같은 많은 컬랙션 클래스들이 class cluster이다.

```
id maybeAnArray = /*…*/;
if ( [maybeAnArray class] == [NSArray class] ){
     // will never be hit
}

id maybeAnArray = /*…*/;
if ( [maybeAnArray isKindOfClass:[NSArray class]] ) {
     // will be hit
}
```

- 코코아 클래스 클러스터 하위 클래스 생성 규칙
    - 하위 클래스는 클래스 클러스터의 추상 기본 클래스를 반드시 상속 해야 한다.
    - 하위 클래스는 자신의 저장공간을 정의해야 한다.
    - 하위 클래스는 상위클래스의 문서에 정의된 메소드의 set을 재정의해야 한다.

## 10. Use Associated Objects to Attatch Custom Data to Existing Classes

```
void objc_setAssociatedObject(id object, void *key, id value, objc_AssociationPolicy policy )

id objc_getAssociatedObject(id object, void *key)

void objc_removeAssociatedObject(id object)
```

```
OBJC_ASSOCIATION_ASSIGN                             // assign
OBJC_ASSOCIATION_RETAIN_NONATOMIC     // nonatomic, retain
OBJC_ASSOCIATION_COPY_NONATOMIC       // nonatomic, copy
OBJC_ASSOCIATION_RETAIN                             // retain
OBJC_ASSOCIATION_COPY                               // copy
```

- 연관 객체는 찾기 어려운 버그를 만들 수 있다.

## 11. Understand the Role of objc_msgSend
- C언어는 정적/동적 바인드 함수 호출을 할 수 있다.
- Objective-C는 동적 바인드를 사용하며 메세지를 전달하는 방식을 사용한다. 이는 모두 실행 시간에 이뤄지며, 앱이 실행되면서 동적으로 변경될 수도 있다.
- 컴파일 시간에는 메세지도 C 함수 호출로 변경된다.

```
// 메세지
id returnValue = [receiverObject messageNameSelector:parameter];

// C 함수 호출로 변경
void objc_msgSend(id receiverObject, SEL messageNameSelector, ...); // ... 부분은 parameter
```

- objc_msgSend 함수는 리시버 클래스의 메소드 목록을 살피며, 해당 메소드를 찾지 못하면 상위 클래스를 찾아 올라간다. 메소드를 못찾으면 메시지 포워딩이 된다.
- objc_msgSend는 각 클래스마다 하나씩 있는 fast map에 캐싱이 된다.(캐싱된 선택자를 다시 호출할 경우는 함수 호출과 속도차이가 많이 나지 않는다)

```
objc_msgSend_stret // 구조체 반환
objc_msgSend_fpret // 부동소수점값 반환
objc_msgSendSuper // 상위 클래스 호출 [super message:parameter]
```

## 12. Understand Message Forwarding
- Objective-C 는 동적 바인딩을 사용하기에 컴파일러 시간에서 클래스에 없는 메세지를 보내는 코드를 에러로 찾을 수 없다.
- 객체가 메세지를 받았을때 메세지를 찾지 못한다면 message forwarding 으로 넘어간다. 이것은 해석할 수 없는 메세지를 처리하는 방법을 개발자가 지정하는 방법이다.

### Dynamic Method Resolution
- 해석할 수 없는 메세지가 전달되었을때 가장 먼저 호출되는 클래스 메소드는 아래와 같다

```
+ (BOOL)resolveInstanceMethod:(SEL)selector // 인스턴스 메소드를 호출할때
+ + (BOOL)resolveClassMethod:(SEL)selector // 클래스 메소그를 호출할때
```

- 선택자를 처리할 수 있는 메소드가 클래스에 추가되어 있는지 여부를 반환. 해당 메소드가 존재하면 다른 포워드 방법으로 넘어가기 전체 구현을 추가할 수 있는 기회를 갖게 된다.

### Replacement Receiver
- 해석할 수 없는 메세지를 처리하기 위해 두번째로 거치는 단계가 다른 리시버가 대체되어 메세지를 처리할 수 있는지 여부를 판단하는 부분이다.

```
- (id)forwardingTargetForSelector:(SEL)selector
```

- 대체 리시버를 반환하거나, 대체 리시버가 없으면 nil 반환
- 메세지를 대체 리시버로 보내기 전에 수정이 필요하면 full forward 방식을 사용해야 한다.

### Full Forward Mechanism
- NSInvocation 객체는 처리되지 않은 메세지의 모든 내용을 감싼다. 여기에는 선택자, 타겟, 인자를 포함한다.

```
- (void)forwardInvocation:(NSInvocation *)invocation
```

- 이 메소드는 호출 타겟을 변경하고, 변경된 것을 호출하는 방법으로 간단히 구현될 수 있다.
- 이 메소드를 구현할때는 처리하지 못한 호출을 해결하기 위해 상위클래스의 구현부분을 항상 호출해야 한다. 마지막에는 NSObject에서 구현된 내용이 호출되고, unhandled selector exception 을 일으키는 doesNotRecognizeSelector: 가 호출될 것이다.

### The Full Picture
- 각 메세지 처리 단계는 이전 단계에서 해결하는것이 비용이 적게 든다. 해결된 메소드는 캐싱이 되고, 같은 선택자의 호출은 포워딩 처리 되지 않는다.
- resolveInstanceMethod -NO-> forwardingTargetSelector -nil-> forwardInvocation -NO->  message not handle

### Full Example of Dynamic Method Resolution
- @dynamic 프로퍼티를 이용해 동적 메소드 해결
- apple 문서 참고 : https://developer.apple.com/library/content/documentation/Cocoa/Conceptual/ObjCRuntimeGuide/Articles/ocrtDynamicResolution.html

## 13. Consider Method Swizzling to Debug Opaque Methods
- Opaque Methods 는 소스 코드를 볼 수 없는 메소드를 말한다. 이것으로 동적 바인딩으로 호출될 메소드를 실행 시간에 바뀔수 있다.
- 소스코드가 없는 바이너리 파일만 있는 경우라도 하위 클래스를 만들거나 메소드를 재정의하지 않고 기능을 변경할 수고, 모든 인스턴스에서 새로운 기능을 사용할 수 있도록 할 수 있는데 이 방법을 method swizzling 이라고 한다.
- 클래스의 메소드 목록은 키가 선택자이고 값이 메소드의 구현인 맵형태로 관리되며, 메소드의 구현은 아래와 같은 프로토타입으로 정의된다.

```
// 메소드 구현은 IMP 라는 함수 포인터로 저장된다.
id (*IMP)(id, SEL, ...)
```

- 런타임에서 몇가지 함수를 이용해 선택자 테이블을 조작할 수 있는데, 이렇게 변경된 메소드 테이블은 바꾼 대상 class의 모든 인스턴스에 사용된다.

```
// 구현 바꿈
void method_exchangeImplementations(Method m1, Method m2)
// 함수 얻기
Method class_getInstanceMethod(Class aClass, SEL aSelector)
```

- 이런 메소드 구현 교환은 디버깅할때 유용할 수 있다.

## 14. Understand What a Class Object Is
- 런타임에 객체 타입을 알아내는 것은 introspection이고, 이는 NSObject 프로토콜에 들어간 강력하고 유용한 기능이다.
- Objective-C 객체 인스턴스는 메모리 blob을 가리키는 포인터이다.
- Objective-C 객체 메모리를 스택에 넣으려고 하면 컴파일 에러가 난다.
- 모든 객체의 메타데이터를 저장하는 데이터 구조체는 id 타입의 정의와 함께 런타임 헤더에 정의된다. 어떤인스턴스의 isa를 호출하면 해당 class의 정보를 준다.
![image](/assets/images/post/effective_obj-c/14.png)

### Inspecting the Class Hierarchy
- isMemberOfClass: -> 객체가 어떤 클래스의 인스턴스인지..
- isKindOfClass: -> 객체가 어떤 클래스의 상속 계층 클래스 인스턴스인지..
- 객체 클래스는 `isa 포인터`, 클래스 상속계층은 `super_class 포인터` 활용
- `object == [someObject class]`와 같이 동등 비교를 할수도 있지만, introspection 메소드를 사용하는 것이 좋다. 메세지 포워딩을 사용한 객체도 비교할 수 있기 때문이다.
- 모든 선택자를 다른 객체로 포워드 하는 객체를 proxy 라 하고, NSProxy는 프록시 객체의 최상위 클래스이다. proxy 객체의 class 메소드를 호출하면 proxy된 객체 클래스가 아닌 NSProxy의 하위 클래스 같은 프록시 클래스가반환된다.

# 3. Interface and API Design
## 15. Use Prefix Names to Avoid Namespace Clashes
- objective-c 는 built-in namespace가 없기때문에 이름에서 충돌이 날 수 있다.
- cocoa를 사용해보면 애플에서 두글자 prefix를 사용중인걸 알수 있기때문에, 3글자 prefix 추천.
- 또다른 잠재적 충돌은 순수 c 함수와 구현파일내 사용된 전역변수이다.

## 16. Have a Designated Initializer
- Designated initializer : 지정 초기화 메소드
- 초기화 메소드가 여러개 있는 것도 괜찮지만, 다른 초기화 메소드에서 호출하는 지정 초기화 메소드는 한개만 있도록 하는게 좋다.
- 하위 클래스에서 지정 초기화 메소드가 있다면, 상위 클래스의 지정 초기화 메소드를 재정의 해야한다.
- 상위 클래스의 초기화 메소드메서드를 사용하지 않는다면, exception을 던지도록 override 할 수 있다.

```
- (id)init
{
    @throw [NSException
            exceptionWithName:NSInternalInconsistencyException
            reason:@"Must use initWithWidth:andHeight: instead."
            userInfo:nil]l;
}
```

## 17. Implement the description Method
- NSObject의 description은 클래스 이름과 객체의 메모리 주소만을 보여주기 때문에 자신이 만든 객체에서는 description을 재정의 하는게 좋다
- 많은 양의 정보는 NSDictionary의 description메소드를 활용하라.

```
- (NSString*)description{
     return [NSString stringWithFormat:@“<%@: %p, %@>“,
                [self class],
                self,
                @{@“title” : _title,
                    @“latitude” : @(_latitude),
                    @“longitude” : @(_longitude}];
}
```

- debuger 안에서 print-object 명령어로 호출되는 debugDescription도 있다.
- LLDB에서 po 명령어로 print-object 실행

## 18. Prefer Immutable Objects
- 필요할 경우에만 객체를 mutable로 만드는걸 추천.
- 프로퍼티 속성을 readonly로 설정하여 immutable하게 만들수 있다.
- 객체 내부에서 데이터를 변경하고 싶다면, 내부적으로 클래스 확장 카테고리를 사용하여 readwrite로 재선언하면 된다.
- 가변객체를 프로퍼티로 노출하는것 보다는 객체를 통해 변경할수 있는 메소드를 제공해라.

## 19. Use Clear and Consistent Naming
- 일관되고 명확하게..
- 메소드와 변수명에는 소문자로 시작하는 카멜 표기법
- 클래스명은 대문자로 시작하는 카멜 표기법, 두세글자의 prefix.

### Method Naming
- 메소드가 새롭게 생성된 값을 반환한다면 맨앞에 수식어가 있는 경우는 제외하고 메소드의 반환타입이 첫번째 단어가 되어야 한다.
- 메소드명의 파라미터는 타입을 나타내는 명사 바로 뒤에 있어야 한다.
- 액션이 있는 메소드에서 액션을 위한 파라미터가 필요하다면, 명사들에 의해 뒤따르는 동사가 포함되어야 한다.
- str같은 약어보단 string
- boolean 타입은  is, has..
- 외부 파라미터를 통해 값을 반환 하는 경우는 위해 get 접두어는 놔둔다..

### Class and Protocol Naming
- prefix
- UIView의 view, Delegate protocol의 delegate같은 단어들로 끝나도록 naming..

## 20. Prefix Private Method Names
- 퍼블릭 메소드와 프라이빗 메소드를 구부하기 위해 프라이빗 메소드에 prefix를 붙이는 것이 좋다.
- objective-c에서는 메소드를 프라이빗으로 선언할 수 있는 방법이 없기때문에 naming이 프라이빗 메소드를 구분할 수 있는 유일한 방법이다. (프라이빗을 따로 선언할수 없기때문에 동적 특성을 같은 이점이 될 수 있다.)
- 애플은 프라이빗 접두어로 언더바 _ 를 잘 사용한다. 이는 충돌 가능성이 있기때문에 피해야 한다. (메소드를 특정 범위내로 한정 짓지 못하는게 단점이긴 하지만, 강력한 동적 메소드 디스패치(dynamic method dispatch) 시스템의 일부이다.)

## 21. Understand the Objective-c Error Model
- ARC 는 exception 안전하지 않다.
- 리소스를 release 하지 전에 exception이 나면 해당 releases는 완료되지 않는다.
- 심각한 에러가 발생했을 경우는 exception을 사용해야 한다.
- objective-c 는 클래스를 추상클래스로 선언할 수 있는 생성자가 없기때문에, 비슷하게 구현하려면 하위 클래스에서 재정의 해야하는 모든 메소드에서 재정의가 안될 경우 예외를 던지도록 하는 것이다.
- 심각하지 않은 일반적인 에러에서는 nil 또는 0을 반환하거나 NSError를 사용한다.

### 일반적인 error설계
- delegate protocol 사용 (ex. NSURLConnectionDelegate)
- 메소드를 호출시 외부파라미터로 NSError 전달

```
- (BOOL)doSomething:(NSError**)error
```

- 에러코드 enum화

## 22. Understand the NSCopying Protocol

```
- (id)copyWithZone:(NSZone*)zone
```

```
- (id)mutableCopyWithZone:(NSZone*)zone
```

- 깊은복사(deep copy) : 객체 내부의 모든 인스턴스 객체 복사
- 얕은복사(shallow copy) : 컨테이너만 복사, 내부 객체를 복사하길 원치 않거나 복사할수 없는 것들이 있는 경우..., Foundation Framework의 모든 컬랙션 클래스는 얕은복사
- 깊은 복사는 따로 정의된 프로토콜이 없고, 내부 객체 복사를 모두 구현해야 한다.
- 가능하면 일반적인 복사에서 얕은 복사

# 4. Protocol and Categories

## 23. Use Delegate and Data Source Protocol for Interobject Communication
- objective-c는 다중상속을 지원하지 않기때문에 protocol을 이용하여 메소드 집할을 정의
- protocol은 delegate 패턴을 구현하기 위해 사용된다.
- category는 상속 없이 메소드를 추가할 수 있는 방법이다. objective-c 런타임이 동적이기 때문에 가능
- delegate 패턴을 이용하면 데이터를 비즈니스로직과 분리 시킬 수 있다. 어떤 데이터를 보여줄지를 관리하는 부분과 데이터 상호작용시 어떻게 처리해야 하는지 결정하는 부분 분리 가능.(data source, delegate로 분리 가능)

### delegate 패턴

- delegate 패턴을 사용시 delegate 프로퍼티는 weak속성으로 정의한다.(retain cycle 방지)
- delegate의 경유 내부적으로만 사용하기 때문에 클래스 확장 카테고리에 선언한다.

```
@implementation EOCDataModel () <EOCNetworkFetcherDelegate>
@end
```

- delegate의 optional method를 호출하기전에 respondToSelector 메소드로 응답할 수 있는지 확인하는 부분이 필요



### data source 패턴
- 데이터를 제공하는데 초점이 맞춰진 protocol

### protocol 메소드 응답 여부 캐싱 (bitfield 구조체)

```
@interface EOCNetworkFetcher(){
     struct{
          unsgined int didReceiveData : 1;
          unsigned int didFailWithError : 1;
          unsigned int didUpdateProgressTo : 1;
     } _delegateFlags;
}
@end

- (void)setDelegate:(id<EOCNetworkFetcher>)delegate {
     _delegate = delegate;
     _delegateFlags.didReceiveData = [delegate respondsToSelector:@selector(networkFetcher:didReceiveData:)];
     ...
}
```

```
if ( delegateFlags.didUpdateProgressTo ){
     [_delegate networkFetcher:self didUpdateProgressTo:currentProgress[;
}
```

## 24. Use Categories to Break Class Implementations into Manageable Segments
- objective-c의 category를 이용하여 클래스를 논리 구성 단위로 나눔 -> 디버깅시 도움
- 프로퍼티와 초기화 메소등의 필수 구현 부분은 메인에 정의하고, 추가적인 메소드 집합을 모아서 카테고리로 나눌 수 있다.
- 카테고리가 많아지면 관리가 어려우므로, 카테고리를 각각의 독립적인 파일로 분리하면 관리하기 좋다.
- private method를 카테고리로 만들어 모아두면 좋다. 이것은 self-documenting이 될 수 있다.
- 카테고리의 헤더가 라이브러리에 배포에 포함되지 않는다면, 외부에서는 카테고리로 되어 있는 private 메소드의 존재를 알 수 없다.

## 25. Always Prefix Category Names on Third-Party Classes
- 카테고리에 구현된 메소드는 런타임시 클래스의 메소드 리스트에 추가된다. 때문에 메소드명이 동일하다면 이미 클래스에 구현된 메소드를 카테고리에서 구현한 메소드가 덮어 쓰게(overriding) 된다.
- 이를 방지하기위해 카테고리 이름과 메소드에 네임스페이스를 붙일 수 있다.

## 26. Avoid Properties in Categories
- 카테고리가 클래스에 인스턴스 변수를 추가하는 것은 클래스 확장 카테고리에서만 가능하다. 때문에 카테고리에서 프로퍼티를 선언하더라도 해당 인스턴스 변수를 synthesize 할 수 없다.
- 카테고리의 프로퍼티에 접근하기 위해서는 별도의 접근자 메소드를 카테고리에 구현하든지, @dynamic 으로 선언해야한다. (@dynamic으로 접근자 메소드를 선언하는 것은 컴파일가 구현을 볼 수 없다는 것...)
- 데이터 캡슐화하려는 프로퍼티 선언은 모두 메인 인터페이스에서 선언되어야한다.

## 27. Use the Class-Continuation Category to Hide Implementation Detail
- 클래스 확장 카테고리는 인스턴스 변수를 선언할 수 있는 유일한 카테고리
- 카테고리 이름 없이 사용

```
@interface EOCPerson ()
// Method here
@end
```

- 클래스 확장 카테고리는 nonfragile ABI 때문에 메서드와 인스턴스 변수를 같은 곳에 정의할 수 있다. 이것은 객체의 크기를 알지 않아도 객체를 사용할 수 있는 것 때문.

```
@interface EOCPerson(){
     NSString *_anInstsanceVariable;
}
// Method declarations here
@end

@implementation EOCPerson {
     int _anotherInstanceVariable;
}
// Method implementation here
@end
```

### objective-c++ 에서 사용
- 클래스 확장 카테고리는 objective-c++ 코드에서 유용함. 해더에서 c++ 클래스를 포함하게 되면 해당 해더는 .mm (objective-c++ 코드를 컴파일할 것이라고 알리는 확장자) 확장자를 갖어야 하지만 클래스 확장 카테고리에 c++ 클래스를 포함하면 해당 해더는 c++로 부터 자유로울 수 있다.

### 내부용 프로퍼티 속성 변경에 사용
- 또 다른 예로는 public interface에 정의된 읽기 전용 프로퍼티를 클래스 내부에서 읽기쓰기 속성으로 변경할 수 있는 것이다. (외부에서는 읽기전용으로 사용, 내부에서는 읽기쓰기 가능)
- 이 방법은 observer가 프로퍼티를 읽는 동시에 내부에서 해당 프로퍼티에 쓰려고 할때 race condition 문제가 발생할 수 있지만, 동기화를 사용하여 해결 가능하다.

```
// public interface
@interface EOCPerons : NSObject
@property (nonatomic, copy, readonly) NSString* firstName;
@property (nonatomic, copy, readonly) NSString* lastName;
- (id) initWithFirstName:(NSString*)firstName
                      lastName:(NSString*)lastName;
@end

// 구현파일내의 클래스 확장 카테고리
@interface EOCPerson()
@property (nonatomic, copy, readwirte) NSString* firstName;
@property (nonatomic, copy, readwirte) NSString* lastName;
@end
```

### 프라이빗 메소드 선언에 사용
```
@interface EOCPerson()
- (void)p_privateMethod;
@end
```

### 객체가 사용하는 프로토콜의 내용을 내부적으로 알려야 할 경우 사용
- delegate등과 같은 프로토콜을 따를 경우 public interface에 프로토콜을 선언하지 않고, 클래스 확장 카테고리안에서 선언하게 되면 해당 클래스가 어떤 프로토콜을 따르고 있는지 외부에 알리지 않고 사용 가능

## 28. Use a Protocol to Provide Anonymous Objects
- 반환되는 객체를 해당 프로토콜을 사용하는 id 타입으로 반환하면, 해당 클래스의 자세한 api 구현 내용을 숨길 수 있다.
- anonymous objects : 익명객체, 다른언어의 개념(이름 없이 인라인으로 클래스를 생성하는 개념)과는 다름

```
@property (nonatomic, weak) id <EOCDelegate> delegate;
```

- 클래스 타입은 상관없이 특정 메소드(protocol)를 사용해야 하는 경우에 익명 객체 활용

### NSDictionary
- NSDictionary 는 익명 객체 개념을 적용했다.

```
-(void)setObject:(id)object forkey:(id<NSCopying>)key
```
