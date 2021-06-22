---
title: "[iOS, Objective-c] Effective Objective-c 2.0 #5~#7"
excerpt: "Effective Objective-c 2.0 #5~#7"
description: "Effective Objective-c 2.0 #5~#7"
last_modified_at: 2018-02-05T14:00:00+09:00
categories: "iOS"
tags: [iOS, Objective-c, Effective Objective-c]

toc_sticky: false

header:
  teaser: /assets/images/teaser/ios-teaser.png
---

# 5. Memory Management
## 29. Understand Reference Counting
- objective-c에서는 reference count를 이용하여 메모리를 관리한다.

### How Reference Counting Works
- 레퍼런스 카운팅 아키텍쳐는 해당 객체가 살아 있는지를 가리키는 카운터를 할당한다. 이 카운트를 retain count / reference count 라고 한다.
- retain : 리테인 카운트 증가
- release :   리테인 카운즈 감소
- autorelease : autorelease pool이 drained될때 감소
- 객체는 리테인 카운트가 1인 상태로 생성되며, release를 통해 0이 되면 dealloc 된다.
- alloc 시 객체가 생성되었다고 해서 리테인 카운트가 1이 아닐수 있다. 반환객체에 대해 내부에서 retain을 여러번 호출했을 경우 해당 객체의 리테인 카운트는 1보다 클 수 있다.
- 객체가 제거되면 그 객체의 메모리가 available pool로 되돌려질뿐이다. 해당 객체의 메모리 주소가 덮어쓰여지지 않았다면 해당 메모리 주소로 접근은 크래쉬가 나지 않을 수 있다. 때문에 객체를 release 하고, 아무도 접근을 하지 못하도록 포인터에 nil값을 주는 코드도 있다.

```
NSNumber *number = [[NSNumber alloc] initWithInt:1337];
[array addObject:number]; // array에서 해당 객체 접근가능
[number release];
number = nil;
```


### Memory Management in Property Accessors
- strong속성으로 정의된 프로퍼티의 값은 리테인될것이다.setter에서 새로운 값은 리테인되고, 예전 값은 릴리즈되며 해당 프로퍼티의 값은 새로운 값으로 교체 된다.

```
- (void)setFoo:(id)foo{
     [foo retain];
     [_foo release];
     _foo = foo;
}
```




### Autorelease Pools
- release를 호출하면서 바로바로 리테인을 줄이는 것 대신에 autorelease를 통해 릴리즈되는 타이밍을 나중으로 밀수 있다.
- 객체를 반환해주는 메소드에서, 객체를 반환해주는 동시에 release를 할 수 없다. 이때 autorelease를 사용하면 해당 메소드를 호출하는 범위 내에서는 반환되는 객체를 보존해주며, 그 이후 release  하게 된다.
- autorelease pool내의 객체들은 event roof가 끝나기 전까지 보존, 이후 release.

### Retain Cycles
- 여러 객체가 서로 참조를 하게되면 순환 참조가 발생하고 메모리 누수가 발생한다.(참조가 순환 관계에 있으면 어느 하나 리테인수가 0이 될 수 없다)
- 이 경우 weak 속성을 사용해 해결한다.

## 30. Use ARC to Make Reference Counting Easier
- Clang 컴파일러 프로젝트는 레퍼런스 카운팅의 문제 부분을 찾을 수 있는 정적 분석기를 도입. 이 정적분석기를 통해 메모리 누수를 찾을 수 있고, 문제가 있는 부분을 알려줄 수 있다. 이를 통해 ARC가 만들어졌다.
- ARC는 개발자 대신 자동으로  retain과 release를 추가해준다. 때문에 메모리 관리 메소드(retain, release, autorelease, dealloc)을 직접 호출할수 없다.
- ARC는 최적화된 로우 레벨의 C함수를 호출하여 메모리를 관리한다.

### Method-Naming Rules Applied by ARC
- 객체를 반환하는 메소드에서 메소드 이름이 alloc, new, copy, mutableCopy와 같이 시작하면 메소드를 호출하는쪽(호출하는쪽에서 객체를 릴리즈할 필요가 있음)에서 소유한 객체로 반환
- 위의 메소드를 제외하고는 호출한 코드쪽에서 반환된 객체를 소유하지 않기때문에, 오토릴리즈되어 반환된다. 메소드 호출 범위 내에서만 살아있게 된다.
- ARC는 성능 최적화를 위해 제거해도 되는 retain, release, autorelease들을 제거한다.
- 메모리 관리를 컴파일러와 런타임에 넘길 수 있기때문에 ARC 성능이 좋아 질 수 있다.

### Memory-Management Semantics of Variables
- ARC는 추가로 지역 변수와 인스턴스 변수의 메모리 관리도 관여한다.
- ARC는 인스턴스 변수 setter에서 새로운 값을 리테인한후에 이전값을 릴리즈하면서 안전하게 인스턴스 변수를 설정한다.

#### <지역변수와 인스턴스 변수의 시맨틱 식별자>
- __strong : default, 리테인
-  __unsafe_unretained : 리테인하지않고 안전하지 않음, 객체할당해제시 포인터 유지
-  __weak : 리테인하지 않지만 안전함, 객체할당해제되면 포인터를 nil로 설정
-  __autorelease : 객체가 메소드에 참조 파라미터로 존달될때 사용됨. 값반환시 오토릴리즈

###
- block에서 사용되는 객체는 자동으로 리테인되가 때문에 순환참조가 발생할 수 있다. __weak 지역변수로 순환을 제거할 수 있다.

```
NSURL *url = [NSURL URLWithString:@"aa"];
EOCNetworkFetcher *fetcher = [[EOCNetworkFetcher alloc] initWithURL:url];
EOCNetworkFetcher * __weak weakFetcher = fetcher;
[fetcher startWithCompletion:^(BOOL success){
    NSLog(@"Finished fetching from %@", weakFetcher.url);
}];
```

### ARC Handing of Instance Variables
- ARC 는 인스턴스 변수의 메모리도 관리하기 때문에 dealloc 메소드를 hooking하여 강한참조의 변수를 모두 릴리즈 시킨다.
- CoreFoundation 객체 or alloc()을 통해 heap에 할당된 메모리에 대해서도 자동 릴리즈(CFRetain/CFRlease 호출 적용)
- ARC가 할당 해재 코드를 추가 하기때문에 dealloc 메소드가 필요 없다.

### Overriding the Memory-Management Methods
- ARC를 사용하기 전에는 메모리 관리 메소드를 재정의할 수 있었지만, ARC를 사용하면 허용되지 않는다.

## 31. Release Reference and Clean Up Observeation State Only in dealloc
- dealloc 메소드는 객체 생애주가안에 리테인 카운트가 0으로 될때 한 번 호출된다. 언제 호출될지는 보장되지 않는다.
- dealloc 은 런타임중 필요시점에 호출될 것이며, 직접 호출해선 안된다.

### dealloc 에서 하는일
- 객체가 소유한 참조들을 release
- non-objective-c 객체들도 release 필요
- observation behavior 를 청소하는 일도 수행(NSNotificationCenter에 등록된 알림 삭제 등등)

###
-  ARC가 아닌 수동으로 메모리 관리시에는 `[super dealloc]` 을 마지막에 호출해야 하지만 ARC는 자동으로 수행.

### 리소스 해제
- dealloc 의 호출 시점을 정의할 수 없기때문에 `파일디스크립터, 소켓, 메모리블록..`등의 리소스들은 사용이 끝나는 시점에 호출할 수 있는 별도의 메소드를 구현해야 된다.
- `open`, `close`등의 메소드 구현으로 소켓 리소스 관리를 할 수 있다.
- 리소스 정리 메소드를 만들어야 하는 다른 이유는 dealloc의 호출없이 앱이 종료되는 등 dealloc이 호출되지 않고 객체가 계속 살아있는 경우가 발생할 수 있기 때문.
- mac os x 와 ios 앱내의 델리게이트에서 앱 종료시 호출되는 메소드에서 꼭 정리가 필요한 객체 정리를 정의할 수 있다

```
- (void)applicationWillTerminate:(NSnotification *)notification // mac os x
- (void)applicationWillTerminate:(UIApplication *)application // ios
```

### 프로퍼티
- dealloc에서 호출을 피해야될 메소드 -> 프로퍼티 접근자
- 재정의된 프로퍼티 접근자가 객체 할당 해제시 안정하게 호출되지 않을 수 있음
- 프로퍼티는 kvo로도 사용 가능하기 때문


## 32. Beware of Memory Management with Exception-Safe Code
- try 안에서 retainede된 객체가 release되기 전에 예외가 발생하면 catchd에서 해당 객체를 처리하여 메모리 누수를 막아야 한다.
- @finally 블록을 사용하여 예외 상황에서 객체 릴리즈 처리를 할수도 있다.(ARC에서는 release를 호출할 수 없기 때문에 수동 메모리 관리시에만 이 방법을 사용할 수 있다)

### ARC 예외 처리
- ARC에서는 예외 상황시 메모리 관리에 대해 안전하게 처리할 수 있는 `-fobjc-arc-exceptios` 라는 기본적인 기능이 있다. 하지만 이 기능은 기본적으로 꺼져 있는 상태이다.
- objective-c는 앱이 종료되는 에러가 발생했을 경우만 예외를 처리하도록 하기 때문에 위 기능이 기본적으로 꺼진 상태
- 하지만 이 기능을 사용시 정리가 필요한 모든 객체를 찾기 위해 많은 양의 코드를 만들어 내고, 런타임 성능에 좋지 않은 영향을 미친다.

## 33. Use Weak References to Avoid Retain Cycles
- 순환을 이루고 있는 객체들은 결국 다른 객체들에 의해 참조가 되지 않고 있는 상태(순환 객체끼리만 서로 참조)가 발생하기 때문에 메모리 누수가 생긴다.
- 참조순환을 해결하는 좋은 방법은 weak 참조를 사용. unsafe_unretained 속성으로도 비소유관계를 표현할 수 있음
- unsafe_unretained의 경우 객체가 할당 해제 되어도 기존 메모리 주소 값이 남아 있기 때문에 해당 객체로 접근시 앱 크러쉬 발생.(assign 속성과 문법적으로 동일하지만 assing속성은 primitive 타입에만 사용되고, unsafe_unretained는 객체에 사용 -> 해당 프로퍼티 값이 안전하지 않다는 명시적 선언)
- weak는 할당 해제시 프로퍼티 값을 nil로 자동 설정. 할당 해제후 접근시 앱크래쉬는 나지 않지만, nil 셋팅으로 인한 부정확한 정보 접근

## 34. Use Autorelease Pool Blocks to Reduce High-Memory Waterline
- autorelease pool이란 어느 시점에 release 될 필요가 있는 객체들의 모임
- pool 이 모두 소진되면 풀안의 모든 객체에 release 메시지 보냄
- 코코아 환경에서 메인스레드와 gcd의 일부분으로 생성되는 스레드들은 모두 autorelease pool을 가지고 있기때문에, 따로 존재 유무를 고려할 필요가 없다.(이 autorelease pool은 각 스레드가 종료되면 pool을 모두 소진한다)
- main.m 에 있는 autorelease pool 블록은 an outer catch-all-pool 을 다루는 것이고, 기술적으로는 필요 없는 항목이다.

### autorelease pool 중첩
* autorelease pool이 중첩된 경우 가장 안쪽의 pool에 추가 되게 된다.
* 중첩은 앱의 최고 메모리 사용량을 제어 할 수 있게 한다.
* for 루프 밖 pool 이 존재시 for 루프가 실행될수록 많은 메모리를 사용하게 된다. 이때 내부에 pool을 추가하게 되면 루프가 실행되는 도중에도 메모리 관리가 되어 사용량이 낮게 유지 될 수 있다.

### autorelease pool stack
*  오토릴리즈 풀은 스택으로 관리되며, 객체가 추가되면 스택에 넣어지고 풀이 모두 소진되면 스택에서 꺼내진다.
* 오토릴리즈 메세지를 받은 객체는 스택의 가장 위 풀에 추가된다.

###
- @autoreleasepool 블록을 사용하면 오토릴리즈 풀의 범위를 지정할 수 있다.

## 35. Use Zombies to Help Debug Memory-Management Problems
- 코코아에는 `zombie`라는 기능으로 메모리 디버깅을 할 수 있다.(런타임중 할당 해제된 객체를 제거하지 않고, 좀비 객체로 전환)
- 좀비 기능 활성화

```
export NSZombieEnabled = “YES"
./app
```

- 좀비에 메세지를 보내면 메시지가 콘솔에 출력되고 앱종료

```
*** -[CFString respondsToSelector:]: message sent to deallocated instance 0x7ff9e9c080e0
```

- 메모리 버그는 ARC를 사용한다 하더라고 발생 가능

### 좀비 클래스 생성
- 좀비 클래스는 객체 클래스의 prefix에 `_NSZombie_` 라는 명칭이 붙이면서 생성되는데, 이것은 실행시간에 해당 객체가 처음으로 좀비화 될때 좀비 클래스가 생성된다,
- isa 포인터를 조작하여 `_NSZombie_`를 상속받는 좀비 클래스로 전환
- 좀시 생성 환경변수가 yes이면 런타임이 dealloc 메소드를 스위즐링하여 좀비클래스로 변경되도록 한다.
- 실제 앱에서는 사용되지 않는다..(디버깅용도이기 때문에 좀비객체로 인한 메모리누수 감안)
###
-  `_NSZombie_`  클래스는 NSObject와 같은 최상위 클래스이기때문에 어떠한 메세지를 보내더라도 모두 완전 포워드 기술로 넘길것이다. 또한 별도의 메소드를 구현하지도 않는다.

## 36. Avoid Using retainCount
- ARC에서는 객체의 현재 리테인수를 얻어오는 `retainCount` 메소드를 폐기 했다. 이를 사용하면 retain, release, autorelease 를 사용하는 것과 같이 컴파일러 에러가 발생한다.

### retainCount not useful
- retainCount 메소드는 특정 시점의 리테인수를 가져온다. 오토릴리스 풀에서 일어날 감소를 포함하지 않음.
- 해당 객체가 ARC 내에서 오토릴리즈를 기다리고있다면, 아래와 같은 코드는 위험하다.

```
while( [object retainCount] ){
 [object release];
}
```

- retainCount는 0을 절대 반환하지않는다. (0이 되면 객체 할당 해제 수행)
- NSSting 등과 같은 객체는 싱글턴객체이다. 싱글턴객체는 리테인수가 절대 바뀌지 않는다.

###
- 리테인수는 개발자의 생각만큼 정확하지 않기 때문에 이를 디버깅 목적으로 사용하기에도 부적합하다.

# 6. Block and Grand Central Dispatch
## 37. Undestand Blocks
- Block 과 GCD 는 멀티스레드의 핵심이다
- 블록은 c, c++, objective-c에 lexical closure를 제공하며, 코드를 전달하는 기법이다.
- GCD는 dispatch queue를 이용하여 쓰레드를 제공하며, 블록은 이 dipatch queue에 들어갈 수 있다.
- GCD는 쓰레드의 모든 스케쥴을 관리하며, 각 큐를 처리하기 위해 시스템 리소스 사용량에 기반을 둔 백그라운드 스레드를 두고 생성,재사용,제거등을 수행한다.

### Block Basics
- 블록은 범위를 공유하며, 다른 함수에 인라인으로 정의된다.

```
return_type (^block_name)(parameter) = ^(parameter){
...
};
```

- 블록의 강력한 기능은 선언된 곳의 범위를 capture 할 수 있는 것이다.
- 기본적으로 블록이 capture한 변수는 변경할 수 없지만, `__block` 수식어로 변경 가능하게 할 수도 있다.
- 블록이 객체 타입 변수를 capture하면 해당 객체를 리테인하게되지만, 블록이 릴리즈되면 객체도 같이 릴리지 된다.
- 블록도 객체로 여겨지며, retain count가 된다. 때문에 블록의 마지막 참조가 없어지면 할당 해제되며, capture된 다른 객체들도 같이 릴리즈 된다.
- 블록이 인스턴스 메소드에서 정의되면, 해당 self객채의 인스턴스 변수를 사용할 수 있다. 여기서 블록은 self 리테인하게되며, 블록이 self로 참조되는 객체에 의해 리테인되면 참조순환이 발생할 수 있다.

### The Guts of a Block
- 블록은 객체이기에 class 객체에 대한 isa 포인터를 갖고 있다.
- 블록의 메모리중 invoke 변수는 해당 블록의 구현이 있는 곳을 가리키는 함수 포인터이다.
- 다음으로 블록의 메모리를 차지하는 부분이 블록이 실행할 정보가 들어가 있는 block discriptor 부분이다.
- 마지막으로 블록 메모리의 마지막부분에는 블록이 capture한 모든 객체들의 복사본이 포함된다.(객체 자체를 복사하지 않고, 포인터 변수만을 복사)

### Global, Stack, and Heap Blocks
- 블록이 정의되면 기본적으로 스택 메모리에 할당되며, 정의된 범위 내에서만 유효하게 된다.
- 블록에 copy 메세지를 보내게 되면 스택에서 힙으로 복사할 수 있다. 이러면 정의된 범위 밖에서도 사용이 가능해진다.
- 전역블록은 블록을 사용할시마다 스택대신 전역 메모리에 정의된다. 전역블록은 절대 할당 해제 되지 않으며(복사가 되지 않음), 이는 효과적인 싱글턴으로 사용 가능하다.

## 38. Create typedefs for Common Block Types
- 블록은 inherent type 으로 변수에 할당 할 수 있다.
- c 의 기능인 type definition으로 정의할 수 있다.

```
typedef int (^EOCSomeBlock) (BOOL flag, int value);

EOCSomeBlock block = ^(BOOL flag, int value){
     // Implementaion
};
```

- 이 경우 ^다음에 있는 blcok_name이 새로운 타입 이름이 된다.

## 39. Use Handler Blocks to Reduce Code Separation
- iOS 앱에서는 특정시간 동안 응답하지 않으면 자동 종료될 수 있다. system watchdog에서 특정 시간 동안 메인 스레드가 중단된 앱은 종료시킨다.
- delegate 프로토콜을 사용하면 비동기 작업에서 완료 이벤트를 전달 받을 수 있다.
- block 을 completion handler로 사용하면 delegate 보다 명확한 api로 사용 가능하다. 직접 객체에 연관될 수 있다.
- 에러 처리를 위한 블록 api를 사용하는 방법은 두가지가 있다.
    1. 두 개의 handler로 성공과 실패 경우 분리
    2. 하나의 completion block으로 성공과 실패 모두 처리
- 하나의 completion block 으로 성공과 실패를 모두 처리하는 방법이 좀 더 유연하게 상황에 대처할수 있기 때문에 선호된다. (애플 api 사용 방법)
- handler block 을 이용한 api 를 설계할때 해당 블록이 들어갈 큐를 블록 api의 파라미터로 전달

## 40. Avoid Retain Cycles Introduced by Blocks referencing the Object Owing Them
- 블록에서 순환 참조가 발생할 수 있다.
- completion handler에서 순환 참조가 발생할 수 있는 참조는 nil로 만들어줘야 한다.
- completion handler가 동작해야지만 순환참조가 깨지는 경우가 있기때문에, completion handler가 실행되지 않는 조건이 발생하면 누수가 발생한다.
- 또한 completion handler를 사용하는 곳에서 블록이 블록을 소유하고 있는 개체를 참조하게 되면 순환 참조가 발생한다.

## 41. Prefer Dispatch Queue to Locks for Synchronization
- 여러 쓰레드에서 동시 접근하는 문제는 해결하기 위해서는 lock을 사용해 동기화를 해야 한다.
- GCD이전의 방법으로는 built-in동기화 블록과 NSLock 객체를 사용하는 방법이 있었다.
- 동기화 블록

```
- (void) synchronizedMethod{
    @synchronized(self){
    // safe
     }
}
```

- NSLock 객체

```
_lock = [[NSLock alloc] init];

-(void) synchronizedMethod{
    [_lock lock];
    // safe
     [_lock unlock];
}
```

- GCD를 사용하여 lock을 효율적으로 사용할 수 있다.
### serial synchronization queue
- 동일한 큐에서 순차적으로 가져오면 동기화 보장

```
_syncQueue = dispatch_queue_create("com.effectiveobjectivec.syncQueue", NULL);

-(NSString*)someString{
     __block NSString *localSomeString;
     dispatch_sync(_syncQueue, ^{
          localSomeString = _someString;
     });
     return localSomeString;
}

-(void)setSomeString:(NSString*)someString{
     dispatch_sync(_syncQueue, ^{
          _someString = someString;
     });
}
```

- 여기서 setter는 동기화가 필요치 않기때문에(리턴 없음) dispatch_async로 바꾸면 더 깔끔하다.
- * dispatch_async는 실행될 블록을 복사해야 하기에 dispatch_sync보다 느릴 수 있다.

### barrier라는 GCD 병렬 queue
- barrier는 큐의 다른 블록과는 베타적으로 수행. 큐가 처리되고 다음 블록이 barrier라면 큐는 현재의 블록이 끝나기를 기다리고, 그 다음 barrier를 수행한다.

```
void dispatch_barrier_async(dispatch_queue_t queue, dispatch_block_t block);
void dispatch_barrier_sync(dispatch_queue_t queue, dispatch_block_t block);
```

### 순차동기화와 barrier 혼용
- 순차큐 사용보다 빨라질수 있음

```
_syncqueue = dispatch_get_global_queue(DISPATCH_QUEUE_PRIORITY_DEFALUT, 0);

-(NSString*)someString{
     __block NSString *localSomeString;
     dispatch_sync(_syncQueue, ^{
          localSomeString = _someString;
     });
     return localSomeString;
}

- (void)setSomeString:(NSString*)someString{
     dispatch_barrier_async(_syncQeue, ^{
          _someString = someString;
     });
}
```

## 42. Prefer GCD to perfomSelector and Friends
- performSelector 메소드를 호출하는 것과 selector를 직접 호출하는 것은 기능적으로 같다

```
[object performSelector:@selector(selectorName)];
[object selectorName];
```

- performSelector 메소드를 사용하면 코드를 매우 유연하게 작성할 순 있지만, 컴파일러 입장에서 실행시간에 어떤 선택자가 수행되는지는 알 수 없다.
- 따라서 ARC를 사용하는 곳에서는 performSelector를 사용하면 컴파일러 경고가 나온다.

```
warning: performSelector may cause a leak because its selector is unknown [-Warc-performSelector-leaks]
```

- 컴파일러는 메소드의 이름조차 모르기 때문에 ARC에서는 위 메소드를 실행하고 릴리즈 하지 않는다. 때문에 메모리 누수가 발생할 수 있다.
- performSelector 메소드는 파리미터를 추가할 수 있고, 해당 선택자 호출을 지연해서 실행하거나 별도의 쓰레드로 실행할 수 있는등 다양한 종류들이 있지만, 사용하는데 제약사항이 많고 잠재적인 버그가 생기기 쉽다.

- GCD와 block을 사용하면 performSelector 메소드의 쓰레드 관련 문제들을 해결할 수 있다.
- 호출 시간 지연 방법

```
// performSelector
[self performSelector:@selector(doSomething)
                withObject:nil
                 afterDelay:5.0];

 // dispatch_after
dispatch_time_t time = dispatch_time(DISPATCH_TIME_NOW,
                                                        (int64_t)(5.0 * NSEC_PER_SEC));
dispatch_after(time, dispatch_get_main_queue(), ^(void){
            [self doSomething];
}];
```

- 메인 쓰레드에서 수행

```
// performSelector
[self performSelectorOnMainThread:@selector(doSomething)
                                        withObject:nil
                                   waitUntilDone:NO];

// dispatch_async
dispatch_async(dispatch_get_main_queue(), ^{
            [self doSomething];
}];
```


## 43. Know When to Use GCD and When to Use Operation Queue
- GCD도 좋지만 시스템 라이브러리의 도구들을 사용하는 것이 좋을 수도 있다.
- GCD는 pure C API 이고, operation queue는 objective-c 객체이다. GCD의 큐에 있는 작업은 블록(가벼운 데이터 구조체)이고, operation queue는 objective-c 객체이기 때문에 조금 더 무겁다. 때로는 객체를 사용하는 이점이 더 좋을 경우도 있다.
- NSBlockOperation 이나 NSOperationQueue의 addOperationWithBlock: 메소드를 사용하는 것은 GCD를 사용하는 것과 비슷해보이는데, NSOperation, NSOperationQueue를 사용하는데 몇가지 이점이 있다.
- Cancelling operations : operation queue는 작업 취소가 간단하다.
- Operation dependencies : 다른 operation에 의존할수 있으며, hierarchy of operaions 를 만들수 있다.(다른 작업 성공후의 수행이 가능)
- Key-Value Observing of operation properties : operation에는 KVO에 적합한 프러퍼티가 있다. 작업이 취소되었는지 확인하는 isCancelled와 끝났는지 확인하는 isFinished 같은 것이 있다. 특정 operaion의 상태 변경 시점들을 알수 있고, 동작중인 업무에 대해 제어를 할 수 있다.
- Operation priorities : GCD는 큐 우선순위가 개별 블록단위가 아닌 전체 큐에 대한 우선순위를 갖지만, operation은 각 작업별 우선순위가 있다.
- Reuse of operations : 클래스에 정의된 정보를 모두 사용할 수 있으며, 이러한 operation 클래스를 코드에서 재사용 할 수 있다.


## 44. Use Dispatch Groups to Take Advantage of Platform Scaling
- GCD는 그룹 작업을 할 수 있는 dispatch groups 기능을 제공한다.
- 이 dispatch group 기능으로 모든 작업들이 끝났을때 콜백을 통해 노티를 받을 수 있는데, 이것은 매우 유용하게 사용된다.
- 이점은 여러 작업이 동시에 병렬도 수행될때 모든 작업이 언제 끝났는지 알수 있는 부분이다.

### 그룹 생성

```
// 그룹 생성
dispatch_group_t dispatchGroup = dispatch_group_create();
```

### 그룹 설정
- dispatch group은 그룹 자체를 식별할수 없기에 해당 task와 그룹을 연관 지을 수 있는 별도의 방법이 필요하다.
1. dispatch_group_async 함수 사용

```
void dispatch_group_async(dispatch_group_t group, dispatch_queue_t queue, dispatch_block_t block);
```

2. enter, leave 의 pair 함수 사용

```
void dispatch_group_enter(dispatch_group_t group);
void dispatch_group_leave(dispatch_group_t group);
```

- enter와 leave의 짝이 맞지 않으면 그룹이 끝나질 않게 된다.

### 그룹 wait
- 끝나기를 기다릴 그룹과 기다릴 시간값(timeout)을 파라미터로 그룹을 기다리는 함수.
- 그룹이 timeout 전에 끝나면 0반환, 그렇지 않으면 0이 아닌값 반환
- timeout에 DISPATCH_TIME_FOREVER 상수를 사용하면 그룹이 끝날때까지 영원히 기다림

```
long dispatch_group_wait(dispatch_group_t group, dispatch_time_t timeout);
```

### 그룹 끝 알림
- wait은 현재 쓰레드가 종료된다.
- wait과 달리 그룹이 종료됐을때 알림을 받을 블록과 해당 블록이 실행될 쓰레드를 파라미터로 추가하여 해당 쓰레드에서 블록으로 그룹 종료 알림을 받는다.

```
vodi dispatch_group_notify(dispatch_group_t gorup, dispatch_queue_t queue, dispatch_block_t block);
```

- GCD는 자동으로 쓰레드를 생성하고 재사용하며, 다수의 블록을 동시에 실행한다.
- 병렬로 실행되는 큐에서 사용하는 쓰레드 수는 GCD가 결정하는 factor에 따르고, 시스템 리소스들의 상황에 기반한다.

- 반복 작업을 수행하는 GCD 함수도 있다.

```
// dispatch_apply 함수
void dispatch_apply(size_t iterations, dispatch_queue_t queue, void(^block)(size_t));

// dispatch_apply 예제
dispatch_queue_t queue = dispatch_get_global_queue(DISPATCH_QUEUE_PRIORITY_DEFAULT, 0);
dispatch_apply(array.count, queue, ^(size_t i){
     id object = array[i];
     [onject performTask];
});

```

- 위의 dispatch_apply는 모든 반복이 끝나기전에 현재 쓰레드를 종료시키며, 그렇기에 파라미터의 블록이 현재의 큐에서 수행하려 한다면 데드락이 발생한다. 백그라운드로 수행되길 원하면 dispatch group을 사용하면 됩니다.

## 45. Use dispatch_once for Thread-Safe Single-Time Code Execution
- 싱글턴 디자인 패턴에서 @synchronized 라는 블록을 사용하여 쓰레드에 대해 안전성을 확보했을 것이다.
- GCD에서는 쓰레드에 안전하게 싱글턴 인스턴스를 구현할 수 있는 기능이 있다.

```
vodi dispatch_once(dispatch_once_t *token, dispatch_block_t block);
```

- 이는 static하거나 global하게 선언된 토큰 변수를 통해 스레드에 안전하게 블록을 처음 한번만 수행되게 할 수 있다. (토큰변수는 매번 완전히 동일한 토큰이어야 하기에 static해야 한다)

```
+(id)sharedInstance{
   static EOCClass *sharedInstance = nil;
   static dispatch_once_t onceToken;
   dispatch_once(&onceToken, ^{
        sharedInstance = [[self alloc] init];
   });
   return sharedInstance;
}
```



## 46. Avoid dispatch_get_current_queue
- GCD에는 현재 어떤 큐가 실행되는지 알수 있는 기능이 있다.

```
dispatch_queue_t dispatch_getcurrent_queue()
```

- 하지만 현재 큐를 확인하기 위해 위 기능을 사용하는 건 바람직하지 않다. 동기화에 사용된 큐가 간접적으로 동일한 큐를 다시 접근하여 사용되지 않는 것만 보장되면 현재의 큐를 확인할 필요가 없어진다.
- 디버깅용으로 사용하는 것은 괜찮다

- 큐는 계층적 구조로 이루어져 있기때문에 단일 큐 객체로 표현하기가 어렵다.(현재 큐 조회시 내부 동기화 큐가 대신 반환될 수 있다.)

- 큐를 비교하기 위해선 GCD의 queue-specific data 함수를 통해 큐를 식별할 수 있도록 설정해야 한다.

```
// 큐 특정 함수
void dispatch_queue_set_specific(dispatch_queue_t queue, const void *key, void *context, dispatch_function_t destructor);

// 사용 예제
dispatch_queue_t queueA = dispatch_queue_create(“com.effectiveobjectivec.queueA”, NULL);
dispatch_queue_t queueB = dispatch_queue_create(“com.effectiveobjectivec.queueB”, NULL);

dispatch_set_target_queue(queueB, queueA);

static int kQueueSpecific;
CFStringRef queueSpecificValue = CFSTR(“queueA”);
dispatch_queue_set_specific(queueA, &kQueueSpecific, (void*)queueSpecificValue, (dispatch_function_t)CFRelease);

dispatch_sync(queueB, ^{
     dispatch_block_t block = ^{
          NSLog(@“No deadLock!!”);
     };

     CFStringRef retrivedValue = dispatch_get_specific(&kQueueSpecific);
     if( retrivedValue ){
          block();
     } else{
          dispatch_sync(queueA, block);
     }
});
```



# 7. The System Frameworks
## 47. Familiarize Yourself with the System Frameworks
- 시스템 프레임워크 없이는 mac os x 와 ios의 앱을 objective-c로 개발할수 없다.
- framework는 동적 라이브러리와 인터페이스로 표현된 헤더 파일들의 코드 집합이다. ios 앱에는 동적 라이브러리를 탑재하는것이 안되기때문에(ios7 버전까지..ios8부턴허용?) 정적라이브러리를 사용한 ios용 서드파티프레임워크도 있다.
- 코코아(코코아터치)는 앱을 만들때 사용되는 프레임워크의 집합이라 할 수 있다.

### Foundation
- 메인 프레임워크는 Foundation이다. 여기에는 NSObject, NSArray, NSDictionary등의 클래스가 포함되어 있다.
- Foundation 프레임워크의 prefix는 `NS` 이며, 이는 NeXTSTEP 이라는 운영체제에서 objective-c가 처음 사용되어 만들어졌다.

### CoreFoundation
- CoreFoundation은 objective-c가 아니지만 앱을 작성시 잘 알고있어야 하며, 이는 Foundation 프레임워크의 기능의 많은 부분을 복제한 C API이다.
- NSString은 Foundation이고, CFString은 CoreFroundation이다. 이들은 toll-free bridging이라는 기능으로 casting될 수 있다.

###
- C 로 작성된 API는 objective-c 런타임으로 바로 전달되어 속도 향상에 이점이 있다. 하지만 ARC는 objective-c객체만 관리되므로 C로 작성된 부분에서는 메모리 관리에 신경써야한다.

### UI 프레임워크
- UI 프레임워크인 AppKit과 UIKit은 각각 Foundation, CoreFoundation위에 만들어진 objective-c 클래스를 제공하며, 여기에는 CoreAnimation, CoreGraphics가 있다.

- CoreAnimation 은 objective-c로 작성되어 있고, QuartzCore 프레임워크의 일부분으로 UI 프레임워크가 그래픽을 랜더링하고 애니메이션을 수행할수 있도록 도구를 제공한다.
- CoreGraphics는 C로 작성되어있고, 2D랜더링에 필수로 필요한 데이터 구조체(CGpoint, CGSize, CGRect..)와 함수를 제공한다.


## 48. Prefer Block Enumeration to for Loops
### For Loops
- for 루프는 일반적으로 정렬되어 있는 컬랙션을 열거하기에는 매우 좋지만, NSDictionary나 NSSet과 같이 정렬되어 있지 않은 컬랙션을 열거하기에는 번거로워진다.(정렬된 키 배열 혹은 값들을 정렬한 배열을 얻을 수 있는 요청 필요)

### Objective-C 1.0 Enumeration Using NSEnumerator
- NSEnumerator는 concrete subclass가 구현할 두개의 메소드를 정의한 abstact bae class이다.

```
-(NSArray*)allObjects
-(id)nextObject
```

- nextObject를 이용하여 다음 순서의 객체를 반환 받는다.

```
NSArray *anArray = /*...*/;
NSEnumerator *enumerator = [anArray objectEnumerator];
id object;
while( (object = [enumerator nextObject]) != nil ){
     // Do something with object
}
```

- reverseObjectEnumerator를 이용하여 거꾸로 컬랙션을 열거 할 수 있다.

### Fast Enumerator (Objective-C 2.0)
- for 루프에서 in 키워드를 사용하여 NSEnumerator를 사용하는 것과 비슷하게 사용

```
NSArray *anArray = /*...*/;
for (id object in anArray ){
      // Do something with object
}
```

- NSFastEnumeration 프로토콜을 이용하여 동작하며, Fast Enumerator를 지원한다는걸 보여주려면 해당 프로토콜을 정의하면 된다.

```
-(NSUInteger) countByEnumeratingWithState:(NSFastEnumerationState*)state
                                                                objects:(id*)stackbuffer
                                                                count:(NSUInteger)length
```

- 이 방법은 굉장히 효율적이지만, 열거형의 인덱스 사용이 쉽지 않을 수 있다.

### Block-Based Enumeration
- 블록을 기반으로 열거하는 방법
- NSArray에 정의된 열거 메소드는 아래와 같다.

```
- (void)enumerateObjectsUsingBlock:(void(^)(id object, NSUInteger idx, BOOL *stop))block
```

- object는 해당 객체, idx는 해당 인덱스, stop은 열거를 중단하는 방법 제공

```
NSArray *anArray = /*...*/;
[anArray enumerateObjectsUsingBlock: ^(id object, NSUInteger idx, BOOL *stop){
     // Do something with object
     if ( shouldStop ){
          *stop = YES;
     }
}];
```

- NSDictionary block enumeration

```
NSDictionary *aDictionary = /*...*/;
[aDictionary enumerateKeysAndObjectsUsingBlock:^(id key, id object, BOOL *stop){
     // Do something with key and object
     if ( shouldStop ){
          *stop = YES;
     }
}];
```

- NSSet block enumeration

```
NSSet *aSet =  /*...*/;
[aSet enumerateObjectsUsingBlock:^(id object, BOOL *stop){
     // Do something with object
     if ( shouldStop ){
          *stop = YES;
     }
}];
```

- NSEnumerationOption 값을 이용하여 병렬작업(NSEnumerationConcurrent), 거꾸로반복(NSEnumerationReverse)..의 설정이 가능

```
-(void)enumerateObjectsWithOptions:(NSEnumerationOptions)options
                                            usingBlock:(void(^)(id obj, NSUInteger idx, BOOL *stop))block

-(void)enumerateKeysAndObjectsWithOptions:(NSEnumerationOptions)options
                                                            usingBlock:(void(^)(id key, id obj, BOOL *stop))block
```

## 49. Use Toll-Free Bridging for Collections with Custom Memory-Management Semantics
- Toll-Free Bridging은 Foundation의 objective-c 클래스와 CoreFoundation의 C 데이터 구조체간의 캐스팅을 쉽게 해준다.
- CoreFoundation의 C 로 짜여진 api 은 objective-c 의 클래스나 객체와는 다르게 데이터 구조체의 형태를 갖고 있다.
- Toll-Free Bridging 예제는 아래와 같다.

```
NSArray *anNSArray = @[@1, @2, @3, @4, @5];
CFArrayRef aCFArray = (__bridge CFArrayRef)anNSArray;
NSLog(@“Size of array = %li”, CFArrayGetCount(aCFArray));
// output : Size of array = 5
```

- 캐스팅 안의 `__bridge`는 ARC가 여전히 objective-c 객체의 소유권을 갖고 있음을 뜻하며, `__bridge_retained`와 같이 사용하는 경우는 ARC가 객체의 소유권을 넘겨줌을 의미한다.
- 위 예제에서는 `__bridge`가 사용되었기때문에 메모리 관리를 위해 별도로 CFRelease(aCFArray)를 호출할 필요없이 ARC 에서 처리된다.
- 반대로 CFArrayRef 를 NSArray * 객체로 캐스팅하고 ARC로 객체의 소유권을 넘기려면 `__bridge_transfer`를 사용하면된다.
- `__bridge`, `__bridge_retained`, `__bridge_transfer` 이 캐스팅 타입을 bridged cast라고 한다.

### Foundation과 CoreFoundation간 전환 필요 상황
- Foundation의 dictionary에서는 키가 복사되고 값이 리테인 되는 부분이 문제가 될 수 있다.
- 키로 사용할 객체에서 객체 복사가 되지 않는다면 NSCopying 프로토콜이 지원되지 않는다는 런타임에러가 발생한다. 이경우 CoreFoundation 단에서 구현하면 메모리 관리 시맨틱을 변경할 수 있다. 즉 키를 복사하지 않고 값을 리테인할 수 있게 된다.

```
#import <Foundation.Foundation.h>
#import <CoreFoundation/CoreFoundation.h>

const void* EOCRetainCallback(CFAllocatorRef allocator, const void *value){
     return CFRetain(value);
}

void EOCReleaseCallback(CFAllocatorRef allocator, const void *value){
     CFRelease(value);
}

CFDictionaryKeyCallBacks keyCallbacks = {
     0,
     EOCRetainCallback,
     EOCReleaseCallback,
     NULL,
     CFEqual,
     CFHash
};

CFDictionaryValueCallbacks valueCallbacks = {
     0,
     EOCRetainCallback,
     EOCReleaseCallback,
     NULL,
     CFEqual
};

CFMutableDictionaryRef aCFDictionary = CFDictionaryCreateMutable(NULL,
                                                                                                                    0,
                                                                                                                    &keyCallbacks,
                                                                                                                    &valueCallbacks);

NSMutableDictionary *anNSDictionary = (__bridge_transfer NSMutableDictionary*)aCFDictionary;
```

- 이와 비슷한 구현으로 배열이나 집합에서도 객체를 리테인하지 않도록 메모리 관리 시맨틱을 변경 할 수 있는데, 만약 배열을 다루는중 리테인 되지 않는 객체가 할당해제되고 배열에서 접근한다면 앱이 크러쉬 날 수 있기에 이런 상황은 주의 해야 한다.

## 50. Use NSCache Instead of NSDictionary for Caches
- 캐싱용도로 사용할때는 NSDictionary 보다는 NSCache를 사용하자

### Benefir of NSCache

- 시스템 메모리가 꽉 차면 자동으로 캐시의 메모리가 정리된다.
- NSDictionary를 캐시 용도로 사용하려면 캐싱하고 있는 메모리가 부족할 경우 직접 메모리를 정리하는 코드를 구현해야 한다.
- NSCache는 최근에 가장 적게 사용된 객체부터 정리를 한다.
- NSDictionary는 키를 항상 복사하여 사용하지만, NSCache는 키값을 리테인하여 한다. 그렇기 때문에 NSCopying을 지원하지 않는 객체라도 키로 사용할 수 있다.
- NSCache는 쓰레드에 안정하기 때문에 여러 쓰레드에서 동시에 사용하기 위해 각각 lock을 걸 필요가 없어진다.

### Use NSCache
- NSCache는 캐시되고 있는 객체들을 제거할 수 있는 방법을 제어 할 수 있다.
    1. countLimit : 캐시의 최대 객체 갯수를 제한
    2. totalCostLimit : 캐시의 전체 cost 제한
- 하지만 객체가 제거될 순서는 특정하게 구현되어 있기에, 사용자가 직접 cost조작하여 임의의 객체를 제거하려고 하는 것은 무의미?하다.(cost가 높다고 무조건 제거가 아니라 특정 구현 방법으로 제거될수도 있다는 의미를 갖게됨)
- NSCache 사용할때 cost limit 계산은 계산을 하는데 있어서 다른 리소스에 무리를 많이 주지 않을떄 수행해야한다.
- 객체를 캐시에 추가할떄마다 cost 계산을 해야하기때문에 cost 계산이 무겁다면 캐시 사용을 보류해야 할 수 있다.
- cost 계산을 위해 NSData객체를 캐시에 추가하면 좋다. NSData는 데이터 크기를 이미 알고 있기때문이다.

```
// 캐시 초기화
NSCache *_cache = [NSCache new];
_cache.countLimit = 100;
_cache.totalCostLimit = 5 * 1024 * 1024;

// 캐시 사용
NSData *cachedData = [_cache objectForKey:url];
if (cachedData)
{
     // cache hit
}
else
{
    //cache miss
    NSData *data = /*...*/;
    [_cache setObject:data forKey:url cost:data.length];
}

```

### NSPurgeableData
- NSPurgeableData는 NSDiscardableContent 프로토콜을 구현한 NSMutableData의 하위 클래스로, 시스템 리소스의 여유가 줄어들면 NSPurgeableData를 사용하는 메모리는 해제될 수 있다.
- NSDiscardableContent 프로토콜의 isContentDiscarded 메소드로 메모리 반납여부를 반환한다.
- 반납될 수 있는 객체에 접근 한다면 NSDiscardableContent 프로토콜의 beginContentAccess 메소드를 통해 반납되는 것을 막을 수 있다. 또한 endContentAccess를 통해 반납을 허용할 수 있다. 이 메소드 호출은 retain count 처럼 중펍 호출되며 참조수가 0이 되야만 메모리 반납을 허용한다.
- NSCache 의 evictsObjectWithDiscardedContent 프로퍼티를 이용하여 메모리가 반납된 객체에 대해 캐시에서도 자동 제거 될지 여부를 설정 할 수 있다.
- 반납될 수 있는 객체 생성시 purge reference count 가 +1 된다. 때문에 endContentAccess를 호출해야만 반납될 수 있다.

## 51. Keep initialize and load Implemetaions Lean
- NSObject 를 상속한 대다수의 클래스에는 초기화 할 수 있는 메소드들이 있다.

### Load

```
+ (void)load
```

- 클래스와 카테고리가 런타임에 추가될때 load 메소드가 딱 한 번 호출되게 된다. 이는 application launch 타이밍에 일어난다.
- mac os x 에서는 dynamic loading 기능이 가능하기 때문에 앱이 실행된 후 라이브러리가 로딩될 수 있다.
- 카테고리의 load의 경우는 해당 클래스의 load가 먼저 실행되고 실행된다.
- load가 실행되는 순간은 런타임이 fragile state 상태가 된다.
- 클래스의 load 메소드는 다른 메소드들보다 먼저 실행됨이 보장된다. 하지만 클래스간에 어떤 클래스 load가 먼저 실행될지는 보장되지 않기에, load 내에서 다른 클래스를 사용하는 것은 안전하지 않다.
- load를 구현하지 않는다면, 상위 클래스에 load가 있어도 호출하지 않는다.
- load가 무겁게 구현된다면 앱이 로딩되는 동안 중단될 수 있다. 그렇기에 load는 최소로 구현해야한다.
- load는 클래스가 수행되기전의 필요 일을 수행하는 곳이 아니라, 디버깅을 위한 곳이다. 또한 카테고리가 성공적으로 로딩됐는지 확인할때 사용될 뿐이다.

### initialize

```
+ (void)initialize
```

- 클래스가 사용되기 전에 딱 한번 호출된다.
- 클래스가 처음 사용되기 바로전에 호출되기 때문에 한번도 사용하지 않은 클래스에서는 호출되지 않는다.
- load는 수행 완료될때까지 앱을 중단 하지만 initialize는 그렇지 않다.
- initialize 가 수행될때는 런타임이 normal state 이다. 때문에 어떤 클래스, 어떤 메소드라도 사용이 가능하다.
- 쓰레드에 대해서도 안전한 환경에서 수행된다.
- load와는 다르게 initialize는 구현되지 않아도 상위 클래스의 initialize가 구현됐다면 실행이 된다.
- initialize의 목적은 내부데이터를 설정하는 것이다.
- static 값 초기화는 컴파일 타임에 할 수 없으므로, initialize에서 해야 한다.

## 52. Remember that NSTimer Retain Its Target
- 아래 메소드를 이용하여 일정 시간 뒤에 이벤트가 발생하는 타이머를 만들 수 있다.

```
+(NSTimer *)scheduledTimerWithTimeInterval:(NSTimeInterval)seconds
                                    target:(id)taget
                                  selector:(SEL)selector
                                  userInfo:(id)userInfo
                                   repeats:(BOOL)repeats
```

- 타이머는 타깃을 리테인 한다. 타이머가 종료(invalidated)되면 타깃을 릴리즈 한다. 타이머가 종료되는 시점은 invalidate가 호출되거나 이벤트가 발생하면이다.
- 타이머가 타깃을 리테인 하기에 반복되는 타이머에서 참조순환 문제가 발생할 수 있다.
- 타이머의 참조순환 문제는 block을 사용하도록 확장하든지(NSTimer 에 카테고리로 확장하여 사용), weak 참조를 사용해서 깨야한다.
