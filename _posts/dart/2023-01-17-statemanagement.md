---
layout: single
title: 플러터 상태관리 파헤쳐보기 
categories:
  - Flutter
tags:
  - [StateMangement]
toc: true
date: 2023-01-16
---

## declarative UI programming (선언형 UI 프로그래밍)

플러터가 UI를 앱의 현재 상태를 반영하여 빌드함.
    
![image](https://user-images.githubusercontent.com/110464205/212729928-51bdf015-f17e-43aa-a80b-64bb86b3c95f.png)
    
예를 들어, 투두리스트에서 할 일을 체크했을 때 처럼, 앱 내 상태가 변했을 때, 그 변화값에 따라 UI를 다시 그림으로써 UI에 반영됌.
    
여기서 imperative (명령형)과의 차이는 (widget.setText) 상태를 변경하면 UI가 처음부터 다시 빌드 된다는 점임. 
    
즉, 선언형은 내가 만든 앱 내에서 상태가 변경됐을 때, 앱 전체 UI가 다시 빌드되는게 아니라(명령형) 바뀐 그 부분만 빌드되는 것에서 차이가 있음.
    

## Differentiate between ephemeral state and app state

### **Ephemeral state** (sometimes called *UI state* or *local state*)
    
하나의 위젯에 깔끔하게 담을 수 있는 state를 말함.
    
```dart
class MyHomepage extends StatefulWidget {
  const MyHomepage({super.key});

  @override
  State<MyHomepage> createState() => _MyHomepageState();
}

class _MyHomepageState extends State<MyHomepage> {
  int _index = 0;

  @override
  Widget build(BuildContext context) {
    return BottomNavigationBar(
      currentIndex: _index,
      onTap: (newIndex) {
        setState(() {
          _index = newIndex;
        });
      },
      // ... items ...
    );
  }
}
```
    
여기서 index라는 field는 앱 내의 다른 부분에서 이 필드를 사용하지 않음. 따라서 MyHomepage 위젯 내에서만 변경됨. 이를 Ephemeral state라 칭함. 
    
내가 정리한 바로는 특정 위젯 내에서만 변경되는 상태.
    

### **App state ( shared state )**

내가 만든 앱 내의 많은 부분에 걸쳐 공유되길 원하는 상태를 App state라고 칭함.

### **There is no clear-cut rule**
    
하나의 위젯 내에서만 쓰이는 상태가 있어도, 앱이 커짐에 따라 그 데이터가 여러 위젯에서 쓰이게 될 수도 있음. 그 의미에서 no clear-cut rule.
    
![image](https://user-images.githubusercontent.com/110464205/212730003-945a242c-556b-41bc-8c48-b49cb2c11a4c.png)
    
따라서 해커와 화가에서 봤듯이,,, 역시 상향식 프로그래밍이 맞는 것 같다. 어떻게 처음부터 모든 걸 패턴으로 파악하고 펼쳐놓고 시작할 수 있을까 …. ! 
    
## **Simple app state management**

![image](https://user-images.githubusercontent.com/110464205/212730044-ec37a8e2-df8b-4e7c-99b1-bba52ffcf2f7.png)

### Lifting state up

플러터에서는 어떤 상태를 사용하는 위젯 위에서 상태를 관리하는 것이 좋음.
    

### Accessing the state

유저가 Catalog에서 아이템을 클릭했을 때 카트에 추가된다. 
    
먼저 콜백함수를 MyListIem에 제공하는 방법이 있음. 
    
```dart
@override
Widget build(BuildContext context) {
  return SomeWidget(
    // Construct the widget, passing it a reference to the method above.
    MyListItem(myTapCallback),
  );
}

void myTapCallback(Item item) {
  print('user tapped on $item');
}
```
    
하지만 이는 ephemeral state 를 관리할 때는 괜찮지만, 이 상태가 여러 위젯에서 쓰이는 경우 수많은 callback들을 다른 위젯들에 전달해야 됌. (매우 별로 ….) 
    
그래서 플러터에서는 이러한 상태들을 관리하는데 편리함을 제공하기 위해 Provider라는 패키지를 제공한다 …. ! 
    

### ChangeNotifier

listener들에게 상태 변경 알림을 제공하는 class임.  
    
어플리케이션의 상태를 캡슐화(encapsulate)하는 한 가지 방법임.
    
ChangeNotifier에 특정한 유일한 코드는 notifyListeners( ) 메서드를 호출하는 것. 앱의 UI가 변경될 수 있을만 한 방식으로 model이 변경될 때마다 이 메서드를 호출.
  
```dart
class CartModel extends ChangeNotifier {
  /// Internal, private state of the cart.
  final List<Item> _items = [];

  /// An unmodifiable view of the items in the cart.
  UnmodifiableListView<Item> get items => UnmodifiableListView(_items);

  /// The current total price of all items (assuming all items cost $42).
  int get totalPrice => _items.length * 42;

  /// Adds [item] to cart. This and [removeAll] are the only ways to modify the
  /// cart from the outside.
  void add(Item item) {
    _items.add(item);
    // This call tells the widgets that are listening to this model to rebuild.
    notifyListeners();
  }

  /// Removes all items from the cart.
  void removeAll() {
    _items.clear();
    // This call tells the widgets that are listening to this model to rebuild.
    notifyListeners();
  }
}
```
    
즉, 내가 관리하고자 하는 상태를 포함한 클래스를 생성한 후 ChangeNotifier 클래스를 상속받는다. 클래스 내부에서 UI가 상태에 따라 변경될 수 있는 메서드 내부에notifyListeners( ) 메서드를 호출한다. 
    

### ChangeNotifierProvider

이는 하위 위젯에 ChangeNotifier의 인스턴스를 제공하는 위젯임.  이 위젯은 그 상태에 접근해야 하는 위젯 위에 위치시킴. 즉, 위의 CartModel 클래스의 경우 MyCart와 MyCatalog 위젯 위에 존재해야 함. 
    
그렇다고 필요 이상으로 상위 위젯에 놓을 필요는 없음. 관리되어야할 부분 위에 놓기. 
    
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

이때 하나 이상의 클래스를 제공하고 싶을 경우, MultiProvider를 사용

```dart
void main() {
  runApp(
    MultiProvider(
      providers: [
        ChangeNotifierProvider(create: (context) => CartModel()),
        Provider(create: (context) => SomeOtherClass()),
      ],
      child: const MyApp(),
    ),
  );
}
```
    
### Consumer

ChangeNotifierProvider를 선언함으로써 앞서 만들었던 CartModel 클래스를 사용할 차례임. 
    
```dart
return Consumer<CartModel>(
  builder: (context, cart, child) {
    return Text('Total price: ${cart.totalPrice}');
  },
);
```
    
사용법은 다음과 같음. 
    
1. Consumer 위젯 옆에 제네릭<>안에 우리가 접근하고 싶은 model의 타입을 명시해야 함.
2. 이 위젯이 꼭 필요로 하는 인수는 builder 임. Builder는 ChangeNotifier가 변경될 때마다 호출되는 함수임.
        
이때 builder는 3가지 인수와 함께 호출되는데
        
        1. context 
        2. ChangeNotifier의 인스턴스(cart) - 이 인스턴스를 통해 cart.totalPrice 처럼 모델 내 데이터를 사용할 수 있음. 
        3. child
    
Consumer 위젯의 best practice는 최대한 내가 쓸 데이터의 변경에 따라 UI가 바뀌길 원하는 부분의 바로 상위 위젯에 사용할 것. 최적화를 위함.
    
```dart
// DON'T DO THIS
return Consumer<CartModel>(
  builder: (context, cart, child) {
    return HumongousWidget(
      // ...
      child: AnotherMonstrousWidget(
        // ...
        child: Text('Total price: ${cart.totalPrice}'),
      ),
    );
  },
);
```

위 처럼 말고, 아래 처럼 사용

```dart
// DO THIS
return HumongousWidget(
  // ...
  child: AnotherMonstrousWidget(
    // ...
    child: Consumer<CartModel>(
      builder: (context, cart, child) {
        return Text('Total price: ${cart.totalPrice}');
      },
    ),
  ),
);
```

### Provider of.

상태를 관리하려고 만든 모델 내 데이터에 실제로 접근할 필요가 없지만 그 클래스 내의 메서드를 사용하려고 하는 경우. Provider.of 메서드 사용.

```dart
Provider.of<CartModel>(context, listen: false).removeAll();
```
