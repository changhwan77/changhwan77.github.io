---
layout: single
title: 무모한 도전 2일차 ㅋ
categories:
  - Dart
tags:
  - [Flutter]
toc: true
date: 2022-12-30
---

### Git 공부

- 새로 브랜치를 만들었는데 remote repo가 설정이 안되...!
- 새로운 기능 추가시 branch 만들어서 push하기(무제한)
    - 즉 코드 수정 전에 어떤 기능을 추가할 지 파악 후
    - 새 branch 만들기
        - 브랜치 명은 추가할 새로운 기능의 특징 또는 이름으로 ! ex)feature/popup_card
- main에 변화가 있으면 브랜치를 main으로 바꿔서 pull

ex) git checkout main →  git pull

- 새 branch에서 작업한 게 있으면 수정한 걸 커밋해놓거나 숨기고 브랜치를 변경해야 됨.
![image](https://user-images.githubusercontent.com/110464205/210083628-63c93a82-69c8-48c8-acc9-36dd055fff8b.png)

### 팝업 메시지 구현

1. showDialog 사용 ⇒ 실패
![image](https://user-images.githubusercontent.com/110464205/210083781-ee9adbf3-6b47-4f21-ab03-4a8322f101f0.png)

2. List<팝업카드> cards = [팝업카드 인스턴스 1];
    - cards[0] 생성 후
    - 팝업 카드의 onPressed 함수 바디에 리스트.clear() 로 리스트를 아예 비워버리는 발상을 했는데 실패 ..

3. Provider를 사용해서 CardModel 클래스 생성해서 removeAll() method 생성해서 2번과 같은 방식으로 시도해봤지만 실패 ! 

4. onDeleted 프로퍼티 사용 ⇒ 실패 !
<iframe src="https://dartpad.dev/embed-flutter.html?id=024241190c4bbb3120021542d225f220" style="width:120%; height:400px"></iframe>
    
![토스 팝업 메시지 실패](https://user-images.githubusercontent.com/110464205/210083378-e7eba7ef-d15b-4e00-9144-40523c35be8f.gif)

### Flutter 공식문서 스터디

**Understanding constraints**

![image](https://user-images.githubusercontent.com/110464205/210083895-abf248d2-1f32-4222-bd97-f3f12090def0.png)

플러터 개발자는 **Constraints go down. Sizes go up. Parent sets position.**

- 제약 조건은 밑으로 크기는 위로 부모 위젯은 포지션을 설정한다 … ?

 **이 규칙을 이해하지 않고서는 진정하게 Flutter의 Layout을 이해할 수 없다 … ?** 

- 위젯은 그 부모 위젯으로부터 자체 제약조건을 가져온다.
    - 여기서 제약조건은 4가지
        - width (가로 길이) 2개 : 최소 / 최대
        - height(세로 길이) 2개 : 최소 / 최대
- 부모 위젯이 children의 각 위젯들에게 제약 조건이 무엇인지 알려주고 각 자식에게 원하는 size를 물음.
- 그런 다음 그 부모 위젯은 children들을 x, y축으로 배치함
- 그러고 자식 위젯들이 부모 위젯들에게 자신의 size를 알려줌 (부모의 제약 조건 내에서)

**Examples**

- Example 9

![image](https://user-images.githubusercontent.com/110464205/210083940-2614406f-2e18-4363-b446-ca9a03a53689.png)

```dart
ConstrainedBox(
  constraints: const BoxConstraints(
    minWidth: 70,
    minHeight: 70,
    maxWidth: 150,
    maxHeight: 150,
  ),
  child: Container(color: red, width: 10, height: 10),
)
```

`Container`가 70 ~ 150 픽셀이라고 추측했는데 틀림. `ConstrainedBox`위젯은 오직 부모 위젯으로 부터 받은 **추가적인** 제약조건만 부과함

여기에서 **화면은 `ConstrainedBox`가 화면과 정확히 같은 크기가 되도록 강제하므로**, 자식 위젯인 `Container`도 화면 전체 크기를 추측 하도록 지시하여 `constraints` 매개변수를 무시한다.

- Example 10

![image](https://user-images.githubusercontent.com/110464205/210083977-1b421050-96b1-4966-8594-1528fa4f038f.png)

```dart
Center(
  child: ConstrainedBox(
    constraints: const BoxConstraints(
      minWidth: 70,
      minHeight: 70,
      maxWidth: 150,
      maxHeight: 150,
    ),
    child: Container(color: red, width: 10, height: 10),
  ),
)
```

`Center` 위젯이 `ConstrainedBox` 위젯에게 스크린 크기를 물어보고 `ConstrainedBox` 위젯이 child 위젯에게 제약 조건을 말해줌.

이 때 Container는 반드시 최소 70, 최대 150 픽셀 사이어야 하는데 10픽셀을 가지고 있으므로 너비와 높이의 최소 값인 70으로 반영돼서 ui가 그려짐

이때 `ConstrainedBox` 위젯에서 정한 제약 조건

- 미달 시 제약 조건의 최솟값
- 초과 시 제약 조건의 최댓값
- 제약 조건 범위 내 들면 자식이 원하는 사이즈

`OverflowBox` 는 스크린의 사이즈를 벗어나도 경고 표시 (노란색 줄) 안띄움

- Example 16

![image](https://user-images.githubusercontent.com/110464205/210084062-85662002-e3a4-4cdc-82dd-f416ef965a52.png)

```dart
UnconstrainedBox(
  child: Container(color: Colors.red, width: double.infinity, height: 100),
)
```

아무것도 렌더링 되지 않음. 

`UnconstrainedBox`가 자식 위젯에게 원하는 사이즈를 말하라고 하는데 Container위젯이 무한 사이즈를 원함.

플러터는 무한 사이즈를 렌더링 할 수 없고 다음과 같은 메시지를 콘솔에 띄움.

`BoxConstraints forces an infinite width.`

- `Flexible(1)`와 `Expanded(2)` 위젯

  - 차이점
    - (1)은 자식 위젯이 자신(1)보다 작거나 같은 너비를 가지도록 함
    - (2)는 자식 위젯이 자신(2)의 너비와 정확히 같도록 함.
  - 공통점
    - 크기를 조정할 때 자식의 너비를 무시함.

**Tight vs. loose constraints**

- **tight 제약조건**
    - 설정한 최대 높이와 너비가 있으면 자식 위젯은 그것보다 크게 사이징
- loose 제약조건
    - 설정한 최대 높이와 너비가 있으면 자식 위젯은 그것보다 작게 사이징


### Getters and setters (아직 덜 이해 됨)

- 어떤 객체의 속성(프로퍼티)를 읽거나 쓸 수 있는 권한을 제공하는 methods.
- `get` and `set` keywords 를 써서 추가적인 프로퍼티를 만들 수 있음.

```dart
class Person {
  String _name;

	String get name => _name;
	set name(String name) => _name = name;
}

main( ) { 

	Person person = Person();
	person.name = 'Kim';
	print(person.name);
}
```

### Simple app state management

**Provider**

3가지 개념 이해 

- ChangeNotifier
- ChangeNotifierProvider
- Consumer

**ChangeNotifier**

- 상태가 변했다는 알림을 listener들에게 알려주는 역할
    - 즉, 상태의 변화를 구독할 수 있음.
- changenotifier를 상속한 클래스 내에 notifyListeners 호출.

**ChangeNotifierProvider**

- ChangeNotifier의 인스턴스를 하위 항목에 제공. provider 패키지 import
- 내가 필요로 하는 상태를 전달 받아야 하는 위젯들의 상위 위젯에 ChangeNotifier 추가

```dart
void main() {
  runApp(
    ChangeNotifierProvider(
      create: (context) => CartModel(),
      child: const MyApp(),
    ),
  );
}
```

**Consumer**

- 관리하려는 상태를 포함한 클래스를 사용할 차례

```dart
return Consumer<CartModel>(
  builder: (context, cart, child) {
    return Text('Total price: ${cart.totalPrice}');
  },
);
```

- 액세스하려는 모델의 type을 지정해야 함.
- builder는 ChangeNotifier가 변할 때마다 호출되는 함수.
    - 즉 model 내 notifyListeners가 호출될 때, Consumer 위젯의 모든 builder 메서드가 호출됨.
- 두번째 인수는 ChangeNotifier의 instance.

### 프로젝트 플랫폼 '플젝윗미'

![image](https://user-images.githubusercontent.com/110464205/210101800-8ac13617-09f6-4b41-9b07-2db70f4bd1bd.png)

장언이에게 아직도 연락오는 창업에 관심있는 친구들,,, 그리고 나는 프로젝트를 해도 기획자, 디자이너와 협업해보고 싶은 마음이 커서 만든 플랫폼이당 

기획, 디자이너, 개발자와 함께 프로젝트르르 진행해보고 싶다면 다음 링크루 ~ 

https://ckdghks9741.oopy.io/ 

### 부족한 점

- 아직 위젯 간 데이터 전달 로직에 약한 것 같다… ㅠ Keep Going !
- 추가로 여기 앱바 관련 UI도 추가적인 공부가 필요 ! 
![image](https://user-images.githubusercontent.com/110464205/210084833-e4057a7a-a6a3-4bb6-89a4-ce45a66a64cc.png)
![app_bar](https://user-images.githubusercontent.com/110464205/210085623-203d7cc2-ae48-438d-ae16-528d2d3e314b.png)
