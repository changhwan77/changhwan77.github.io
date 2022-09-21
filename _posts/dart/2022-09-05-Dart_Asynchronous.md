---
layout: single
title: < Dart Deep Dive > Future, async, await 
categories:
  - Dart
tags:
  - [Flutter, Future]
toc: true
date: 2022-09-05
---

![flutter future](https://navoki.com/wp-content/uploads/2019/10/future-min.png)
출처: Navoki

플러터는 단일 스레드(single-threaded) 언어인 다트를 사용하여 작성합니다. 단일 스레드는 한 번에 한 가지의 작업만 수행할 수 있는데, 이는 플러터 앱이 한 번에 한 가지 작업만 수행할 수 있음을 의미합니다.  

그렇다고 해서 플러터 앱이 동시에 여러가지 작업을 못하는 것은 아닙니다 ! 

## 플러터는 이벤트 루프를 사용합니다.
---
 앱이 시작되고 앱이 멈추는 사이에 사용자들이 계속 탭을 누르는 등  수많은 이벤트가 발생할 것입니다.

 앱은 사용자들이 언제 탭을 눌러 어떤 순서로 이벤트들을 발생시킬지 예측할 수 없고 단일 스레드로 이러한 모든 이벤트들을 처리해야 합니다. 그래서 앱은 이벤트 루프를 실행합니다. 이벤트 큐에서 가장 오래된 이벤트를 가져와서 처리하고 이 과정을 이벤트큐가 비워질 때까지 반복합니다. 

앱이 실행되는 동안(사용자가 폰 화면을 터치하거나, 뭔가를 다운로드하는 등) 이벤트 루프는 이러한 이벤트들을 한 번에 하나씩 처리하면서 돌아가고 있습니다.

![event loop](https://user-images.githubusercontent.com/110464205/188399602-40d66d87-dced-452b-b6d5-b1ae42fdbc05.jpg)


## 한 이벤트가 작업량이 많아서 느려도 기다릴 필요가 없습니다.
---
다트(Dart)는 다음과 같은 방식으로 이를 처리합니다. 

1. 처리가 오래걸리는 이벤트는 일단 시작됩니다. 
2. HTTP 요청, 파일 읽기 등과 같이 무엇인가를 기다리는 순간 다른 이벤트를 처리하러 갑니다.
3. 일종의 리스너(listener)가 형성됩니다. 이 리스너는 기다리고 있는 활동을 모니터링하고 기다리는 것이 끝나면 알려줍니다. 
4. 이 리스너를 참조하고 있던 객체가 메인 스레드(main thread)로 반환되는데, 이 객체가 우리가 배울 *Future*입니다.
5. 그 대기 중인 활동이 끝났을 때, 이벤트 루프가 이를 보고 메인 스레드에서 그 활동과 관련된 메서드(method)를 실행하여 그 느린 이벤트를 마무리하는 것을 처리합니다. 

긴 호흡인 것 같지만 우리가 할 일은 Future라는 코드를 쓰는 것 밖에 없습니다.

```dart
//  ReadaFile()은 처리하는데 시간이 오래걸리는 Method입니다.
Future myFuture = ReadaFile();
// 일단 이 파일을 읽는 메서드를 호출하여 파일 읽기를 시작한 후에 파일로부터 데이터를 받는 동안 
// Dart는 다른 이벤트를 처리하러 갑니다.
```

Future는 지금은 없지만 <span style="color:orange">미래</span> 어느 시점에 요청한 데이터 혹은 에러가 담길 박스라고 생각하시면됩니다.  <U>즉, 요청한 작업의 결과를 기다리지 않고 바로 다음 작업으로 넘어가고 그 후 작업이 완료되면 결과를 받는 방식이라고 보시면 됩니다.</U>

**플러터에서 Future를 사용해서 데이터를 처리하는 방법**
---
보통 콜백이라는 함수를 등록하여 Future에 데이터가 준비되면 그 데이터로 무엇을 할지 알려줍니다. 

**myFuture.then(myCallback);**

.then() 함수는 콜백 함수를 등록하는 방법이고 콜백 함수는 받기로 약속된 데이터를 처리하도록 정의되어야 합니다.  예를 들어서 Future<int>.then 인 경우 콜백 함수는 다음과 같이 정의되어야 합니다. 

```dart
void myCallback(int theIncomingData) {
 doSomethingWith(theIncomingData);
}
```

다음과 같이 앱 화면에 네트워크 요청을 시작하는 버튼이 있다고 가정해 보겠습니다.

```dart
RaisedButton( // (1)
  child: Text('Click me'),
  onPressed: () { // (2)
    final myFuture = http.get('https://예시.com');
    myFuture.then((response) { // (3)
      if (response.statusCode == 200) {
        print('데이터 받기 성공!!');
      }
    });
  },
)
```

(1) 앱을 실행하면 플러터가 Click me라고 적힌 버튼을 화면에 표시합니다.<br/>
(2) onPressed는 사용자가 이 버튼을 클릭하면  'https://예시.com' 같은 네트워크 데이터를  요청하게 되고 <br/>
(3) .then() 함수는 그 데이터를 받아서 그 데이터가 .then() 함수의 바디 {  } 내에 정의된 조건에 부합하면 데이터 받기 성공!! 을 콘솔에 표시합니다. <br/>

## Dart에서 더 직관적으로 이해해봅시다
---
다음은 다트가 실행하는 코드 흐름입니다. 
  
<iframe src="https://dartpad.dev/embed-dart.html?id=fe444d0bd811a5e1471970d3f8abd1ad" style="width:120%; height:400px"></iframe>

Line 7 : fetchUserOrder 메소드를 호출. (카페 직원이 고객의 주문을 받으려고 기다리고 있다고 생각하시면 됩니다.)<br/>
Line 3 : fetchUserOrder 메소드는 2초 후에 문자열 Americano를 결과값으로 내놓을 future 박스를 만든 뒤 바로<br/>
Line 8 : 콘솔에 ‘고객의 주문을 받고 있습니다… 를 표시<br/>
<span style="color:orange">**중요한 것은 fetchUserOrder의 결과값을 받으려면 2초가 걸리기 때문에 이를 기다리지 않고 Line 8번 코드를 바로 실행하는 것입니다.**</span><br/>
Line 7 : 2초 후 future 박스 안에 들어있던 문자열 Americano를 콘솔에 표시하면서 본문 종료.<br/>

## **async와 await**

async 및 await 키워드는 비동기 함수를 정의하고 결과를 사용하는 선언적 방법을 제공합니다. 

### async 및 await를 사용할 때 두 가지 기본 가이드라인

1. **async 함수를 정의하려면, 함수 바디 앞에 async 키워드를 추가.** 

```dart
**void main( ) async { …. }**
```

반환할 값이 선언된 함수라면 Future<Type>로 변경.

```dart
**Future<Type> main( ) async { …. }**
```

2. **await 키워드는 async 함수에서만 작동함.**

async 함수가 있으면 await 키워드를 사용하여 future 함수가 완료될 때까지 기다릴 수 있음

```dart
 **print(await Order( ) );**
```
<br/>
**다음 두 가지 예시의 차이점을 주목해보세요!** 

#### 1) 동기식 함수 

```dart
String createOrderMessage() {
  var order = fetchUserOrder();
  return 'Your order is: $order';
}

Future<String> fetchUserOrder() =>
    Future.delayed(const Duration(seconds: 2), () => '바닐라라떼');

void main() {
  print('주문을 받고 있습니다..');
  print(createOrderMessage());
}
```
![동기식](https://user-images.githubusercontent.com/110464205/188399316-ca4e52a6-f814-4438-b936-cc9ebde4f571.png)

간단하게 설명하자면, 

Line 3 에서 $order에  `fetchUserOrder` 메소드의 리턴 값을 받는데, Line 2의 `fetchUserOrder` 메소드를 실행한 후 2초가 지나지 않았기 때문에 Instance of Future<String>을 반환하는 것입니다. (uncompleted future)

#### 2) 비동기식 함수

```dart
Future<String> createOrderMessage() async {
  var order = await fetchUserOrder();
  return 'Your order is: $order';
}

Future<String> fetchUserOrder() =>
    Future.delayed(const Duration(seconds: 2), () => '바닐라라떼');

Future <void> main() async {
  print('주문을 받고 있습니다..');
  print(await createOrderMessage());
}
```
![비동기식](https://user-images.githubusercontent.com/110464205/188399433-244ef733-d73a-40ec-86ae-3dcca57e9e07.png)

**Line 2: 동기식 실행과 다른 점은 `await` 키워드 때문에 `fetchUserOrder` 메소드가 리턴값을 반환해야 다음 코드로 넘어갑니다. 즉,  ‘바닐라라떼’ 라는 리턴값을 받고 Line 3 코드를 실행하기 때문에 콘솔에 Your order is: 바닐라라떼 가 출력됩니다.**

== 두 예시의 차이점은 다음과 같이 3가지 입니다: ==

- <span style="color:orange">'createOrderMessage()'</span>의 반환 타입이 String 에서 <span style="color:orange">Future<String></span>으로 바뀐 점
- **async** 키워드가 <span style="color:orange">'createOrderMessage()'</span>와 `main()` 함수 바디 {  } 앞에 명시된 점
- **await** 키워드는 비동기 함수 <span style="color:orange">`fetchUserOrder`() 및 `createOrderMessage()`</span> 앞에 명시해야 한다는 점

## **async와 await가 있을 때 실행 흐름**

<iframe src="https://dartpad.dev/embed-dart.html?id=ca77d01189c178cad466cc3aa3a2b4ab" style="width:120%; height:400px"></iframe>  
  
async 함수에서 첫 번째 await 키워드까지는 동기적으로 실행됩니다.

즉, async 함수 본문 내에서 첫 번째 await 키워드 이전의 모든 동기 코드가 즉각적으로 실행됨을 의미합니다. 

위 예시를 실행해본 다음, 다음과 같이 line2와 line3을 바꿔서 실행해보세요 ! 

```dart
var order = await fetchUserOrder();
print('고객의 주문을 기다리는 중...');
```

콘솔에 출력되는 코드의 순서가 달라진 것을 볼 수 있으실 겁니다. await 키워드로 인해 fetchUserOrder() 함수의 결과값인 바닐라라떼가 반환되어야 고객의 주문을 기다리는 중…. 이라는 코드가 출력됩니다. 

여기까지 플러터(Flutter)에서 오래걸리는 작업을 어떻게 처리하는지를 간단히 살펴보고 이를 다루는 다트(Dart)의 클래스(Class)인 Future에 대해 분석해보았습니다! 

혹여나 살펴보면서 잘 모르는 부분이나 궁금하신 점이 있으시다면 편하게 댓글 남겨주시면 감사하겠습니다:)

## 참고 문헌
[Asynchronous programming: futures, async, await](https://dart.dev/codelabs/async-await)<br/>
[dart:async library - Dart API](https://api.dart.dev/stable/2.17.6/dart-async/dart-async-library.html)<br/>
[Futures, async, await: Threading in Flutter](https://medium.com/flutter-community/futures-async-await-threading-in-flutter-baeeab1c1fe3)<br/>
[Dart asynchronous programming: Futures](https://medium.com/dartlang/dart-asynchronous-programming-futures-96937f831137)
