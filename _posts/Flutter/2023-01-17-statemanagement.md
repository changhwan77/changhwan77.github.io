---
layout: single
title: 플러터 상태관리 Deep Dive
categories:
  - Flutter
tags:
  - [StateMangement]
toc: true
date: 2023-10-05
---

앱에서 현재 식단 데이터와 날짜 데이터를 사용하고 있는데 과연 내가 상태 관리에 대해 자신 있게 설명할 수 있을까? 라는 물음을 던졌을 때, No! 라는 대답이 나왔기 때문에 플러터의 상태 관리에 대해 제대로 탐구하기 위해 스터디를 하는 시간을 가졌다. 

## **Start thinking declaratively**

If you’re coming to Flutter from an imperative framework (such as Android SDK or iOS UIKit), you need to start thinking about app development from a new perspective.

Many assumptions that you might have don’t apply to Flutter. For example, in Flutter it’s okay to rebuild parts of your UI from scratch instead of modifying it. Flutter is fast enough to do that, even on every frame if needed.

Flutter is *declarative*. This means that Flutter builds its user interface to reflect the current state of your app:

*declarative(선언형)이란*  Flutter가 앱의 현재 상태를 반영하기 위해 사용자 인터페이스를 구축한다는 뜻.

![상태관리 스터디 1](https://github.com/changhwan77/changhwan77.github.io/assets/110464205/d068f39c-67b2-48a6-ae50-c90fa6116afb)

When the state of your app changes (for example, the user flips a switch in the settings screen), you change the state, and that triggers a redraw of the user interface. There is no imperative changing of the UI itself (like `widget.setText`)—you change the state, and the UI rebuilds from scratch.

Read more about the declarative approach to UI programming in the [get started guide](https://docs.flutter.dev/get-started/flutter-for/declarative).

The declarative style of UI programming has many benefits. Remarkably, there is only one code path for any state of the UI. You describe what the UI should look like for any given state, once—and that is it.

At first, this style of programming might not seem as intuitive as the imperative style. This is why this section is here. Read on.

여기서 명령형(imperative)와 선언형(declarative) 접근 방식에 대한 이해가 되지 않아 추가로 스터디를 진행했다.

## **Introduction to declarative UI**

### **Why a declarative UI?**

Frameworks from Win32 to web to Android and iOS typically use an imperative style of UI programming. This might be the style you’re most familiar with—where you manually construct a full-functioned UI entity, such as a UIView or equivalent, and later mutate it using methods and setters when the UI changes.

Win32부터 웹, 안드로이드, iOS 등의 프레임워크는 명령 중심적 UI 프로그래밍 스타일을 사용한다. 여기서는 완전한 기능의 UI 개체(UIview와 같은)를 수동으로 생성하고 ui가 변경될 때 메서드나 setter등을 사용하여 나중에 변경한다. 

여기서 `setter`란 객체의 속성을 설정하거나 변경하는 메서드를 가리킨다.

In order to lighten the burden on developers from having to program how to transition between various UI states, Flutter, by contrast, lets the developer describe the current UI state and leaves the transitioning to the framework.

다양한 UI 상태 간의 전환 방법을 프로그래밍해야 하는 개발자의 부담을 줄이기 위해, Flutter는 대조적으로 개발자에게 현재 UI 상태를 설명하도록 허용하고 전환 작업은 프레임워크에게 맡긴다. 

This, however, requires a slight shift in thinking for how to manipulate UI.

그러나 이것은 UI를 조작하는 방법에 대한 약간의 사고 변화가 필요하다.

### **How to change UI in a declarative framework**

Consider a simplified example below:

![선언형 프로그래밍 예시1](https://github.com/changhwan77/changhwan77.github.io/assets/110464205/e47b5c6e-9161-444b-a85e-d6f44c790acf)

In the imperative style, you would typically go to ViewB’s owner and retrieve the instance `b` using selectors or with `findViewById` or similar, and invoke mutations on it (and implicitly invalidate it). 

명령형 스타일에서는 일반적으로 ViewB의 소유자로 이동하여 selector나 `findViewById` 또는 유사한 방법을 사용하여 인스턴스 `b`를 검색하고 변경 사항을 적용한다.

invoke란 메서드나 함수를 호출하는 것을 의미하는데, 해당 메서드나 함수가 실행되며 그 결과를 반환하거나 원하는 작업을 수행한다. 

implicitly invalidate it 란 어떤 객체나 상태가 변경되어 이전의 데이터가 더 이상 유효하지 않음을 의미한다. 

For example:

```dart
// Imperative style
b.setColor(red)
b.clearChildren()
ViewC c3 = new ViewC(...)
b.add(c3)
```

You might also need to replicate this configuration in the constructor of ViewB since the source of truth for the UI might outlive instance `b` itself.

설정 정보(컬러 등)가 View B의 인스턴스인 `b` 와 함께 오랜 시간 동안 유지되어야 할 수 있는 상황을 고려하여 생성자에서 이 정보를 복제하여 저장하는 것이 좋다는 의미다. 근데 이렇게 생성자로 따로 계속 설정해서 관리하다 보면 찾기가 정말 불편할 것 같다는 생각이 들긴 했다.

In the declarative style, view configurations (such as Flutter’s Widgets) are immutable and are only lightweight “blueprints”. To change the UI, a widget triggers a rebuild on itself (most commonly by calling `setState()` on StatefulWidgets in Flutter) and constructs a new Widget subtree.

선언적 스타일에서는 뷰 구성 요소(예: 플러터의 위젯)은 불변(immutable)하며 오직 가벼운 “설계도”만 역할을 한다. UI를 변경하려면 위젯이 자체적으로 rebuild하도록 trigger하고(주로 플러터에서 StatefulWidgets의 `setState()`를 호출함으로써) 새 위젯 하위 트리를 구축한다.

```dart
// Declarative style
return ViewB(
  color: red,
  child: const ViewC(),
);
```

Here, rather than mutating an old instance `b` when the UI changes, Flutter constructs new Widget instances. The framework manages many of the responsibilities of a traditional UI object (such as maintaining the state of the layout) behind the scenes with RenderObjects. RenderObjects persist between frames and Flutter’s lightweight Widgets tell the framework to mutate the RenderObjects between states. The Flutter framework handles the rest.

여기서 UI가 변경될 때 이전 인스턴스 `b`를 변경하는 대신, 플러터는 새 위젯 인스턴스를 작성한다. 프레임워크는 RenderObjects를 통해 (레이아웃의 상태를 유지하는) 전통적인 ui 객체의 많은 책임을 내부에서 처리한다. RenderObjects는 프레임 간에 지속되며 플러터의 가벼운 위젯은 RenderObjects에게 상태를 변경하라고 프레임워크에게 알림. Flutter 프레임워크가 나머지를 처리함. 

### 잠시! RenderObject에 대해 알아보자

RenderObject란 건축 작업자라고 생각하면 된다. 예로 들어, 우리는 건축 계획(Blueprint)을 가지고 있고, 이 건축 계획에는 집의 구조와 배치에 대한 모든 정보를 포함하고 있다. 이는 위젯 트리와 비슷한 개념이다. 이 건축 계획을 실제로 구현해야 하는데 건축 작업자(RenderObject)에게 이 계획을 제공한다. 실제로 RenderObject는 계획에 따라 벽, 지붕, 창문 등을 구성하고 배치한다. 

즉, 플러터의 RenderObject가 위젯 트리를 사용하여 화면에 UI를 그리고 배치하는 방식이다.

**결론!**

결론적으로 명령형(Imperative) 프로그래밍 방식은 이전에 생성한 인스턴스`b`를 수정하려면 그 인스턴스 b를 찾아서 정보를 재수정하는 방식이었다면 선언형(declarative)인 플러터에서는 설계도 자체를 가진 위젯의 기본 사항을 변경하여 새 인스턴스를 작성한다. 

## **Differentiate(구분) between ephemeral(일시적인) state and app state**

*This doc introduces app state, ephemeral state, and how you might manage each in a Flutter app.*

In the broadest possible sense, the state of an app is everything that exists in memory when the app is running. This includes the app’s assets, all the variables that the Flutter framework keeps about the UI, animation state, textures, fonts, and so on. While this broadest possible definition of state is valid, it’s not very useful for architecting an app.

가장 일반적인 의미에서 앱의 상태는 앱이 실행 중일 때 메모리에 있는 모든 것을 말한다. 여기에는 앱의 에셋, 플러터 프레임워크가 UI에 대해서 유지하는 모든 변수, 애니메이션 상태, 텍스쳐, 폰트 등이 포함된다. 가장 일반적인 상태 정의는 유효하지만, 앱의 설계하는 데는 그리 유용하지 않다. 

First, you don’t even manage some state (like textures). The framework handles those for you. So a more useful definition of state is “whatever data you need in order to rebuild your UI at any moment in time”. Second, the state that you *do* manage yourself can be separated into two conceptual types: ephemeral state and app state.

먼저, 일부 상태(텍스쳐 같은)는 아예 관리하지 않는다. 프레임워크가 대신 처리하기 때문에 상태에 대한 더 유용한 정의는 “아무 때나 UI를 다시 구축하기 위해 필요한 데이터”이다. 

두 번째로, 당신이 직접 관리하는 상태는 개념적으로 두 가지 유형으로 나눌 수 있다: *일시적 상태와 앱 상태.*

### Ephemeral state

Ephemeral state (sometimes called *UI state* or *local state*) is the state you can neatly contain in a single widget.

일시적 상태(때로는 *UI 상태* 또는 *로컬 상태*라고도 함)는 하나의 위젯 내에서 깔끔하게 포함할 수 있는 상태이다. 

This is, intentionally, a vague definition, so here are a few examples.

- current page in a `[PageView](https://api.flutter.dev/flutter/widgets/PageView-class.html)`
- current progress of a complex animation
- current selected tab in a `BottomNavigationBar`
    
    모호한 정의이므로, 다른 몇 가지 예를 들어보자면 PageView 위젯의 현재 페이지, 복잡한 애니메이션의 현재 상황, `BottomNavigationBar`에서 현재 선택된 탭 등이 있다.
    

Other parts of the widget tree seldom need to access this kind of state. There is no need to serialize it, and it doesn’t change in complex ways.

위젯 트리의 다른 부분은 이 유형의 상태에 접근할 필요가 거의 없다. 이것을 직렬화할 필요가 없으며 복잡한 방식으로 변하지 않는다.

잠시! serialize(직렬화) 라는 것은 데이터나 객체를 일련의 바이트 또는 텍스트로 변환하여 저장하거나 전송할 수 있는 형식으로 변환하는 프로세스를 의미한다. 

byte는 컴퓨터에서 데이터를 저장하고 처리하는 데 사용되는 가장 작은 단위이다. 바이트는 8개의 bit로 이루어져 있으며, bit는 0또는 1의 값을 가질 수 있다. 따라서 하나의 바이트는 2^8 또는 256가지 다른 값을 나타낼 수 있다. 

In other words, there is no need to use state management techniques (ScopedModel, Redux, etc.) on this kind of state. All you need is a `StatefulWidget`.

즉, 이 유형의 상태에 대해 상태 관리 기술을 사용할 필요가 없다. 당신이 필요한 것은 `StatefulWidget` 뿐이다.

Below, you see how the currently selected item in a bottom navigation bar is held in the `_index` field of the `_MyHomepageState` class. In this example, `_index` is ephemeral state.

아래에서 보듯이 하단 네비게이션 바에서 현재 선택된 항목이 `_MyHomepageState`클래스의 `_index` 필드에 저장되어 있다. 이 예제에서 `_index` 는 일시적인 상태이다.

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

Here, using `setState()` and a field inside the StatefulWidget’s State class is completely natural. No other part of your app needs to access `_index`. The variable only changes inside the `MyHomepage` widget. And, if the user closes and restarts the app, you don’t mind that `_index` resets to zero.

여기서 `setState()` 를 사용하고 StatefulWidget의 State 클래스 내부의 필드를 사용하는 것은 완전히 자연스럽다. 앱의 다른 부분에서 `_index` 에 접근할 필요가 없다. 이 변수는 오직 `MyHomepage` 위젯 내에서만 변경된다. 그리고 사용자가 앱을 닫고 시작하면, `_index` 가 0으로 재설정되어도 문제가 없다. 

**즉, Ephemeral(일시적) 상태는 하나의 위젯에서만 깔끔하게 관리해도 되는 데이터로 다른 위젯에서 전혀 사용하지 않는(공유되지 않는) 데이터를 의미한다.** 

### App state

State that is not ephemeral, that you want to share across many parts of your app, and that you want to keep between user sessions, is what we call application state (sometimes also called shared state).

일시적이지 않고 앱의 여러 부분에서 공유하며 사용자 세션 간에 유지하려는 상태를 우리는 앱 상태(때로는 공유 상태)라고 한다.

잠깐! Session은 두 대 이상의 컴퓨터나 디바이스가 서로 대화하고 정보를 주고받을 수 있는 제한된 시간 동안의 상호 작용을 의미한다. 예시로는 전화 통화를 생각해보면 된다!

Examples of application state:

- User preferences(사용자 설정)
- Login info(로그인 정보)
- Notifications in a social networking app(소셜 네트워킹 앱의 알림)
- The shopping cart in an e-commerce app(전자 상거래 앱의 쇼핑 카트)
- Read/unread state of articles in a news app(뉴스 앱의 읽음/읽지 않음 상태)

For managing app state, you’ll want to research your options. Your choice depends on the complexity and nature of your app, your team’s previous experience, and many other aspects. Read on.

앱 상태를 관리하기 위해, 여러 옵션을 조사해야 할 것임. 앱의 복잡성과 특성, 팀의 이전 경험 및 다른 측면에 따라 선택이 다를 것이다.

### There is no clear-cut rule(명확한 규칙은 없다)

To be clear, you *can* use `State` and `setState()` to manage all of the state in your app. In fact, the Flutter team does this in many simple app samples (including the starter app that you get with every `flutter create`).

명확히 하자면, 앱 내의 모든 상태를 관리하기 위해 `State` 와 `setState()` 를 사용할 수 있다. 실제로 플러터팀에서는 많은 간단한 예제에서 이를 사용한다.

It goes the other way, too. For example, you might decide that—in the context of your particular app—the selected tab in a bottom navigation bar is *not* ephemeral state. You might need to change it from outside the class, keep it between sessions, and so on. In that case, the `_index` variable is app state.

반대로도 가능하다. 예를 들어 특정 앱의 컨텍스트에서 하단 네비게이션 바에서 선택한 탭이 일정한 상태가 아닐 수 있다고 결정할 수 있다. 이를 클래스 외부에서 변경하거나 세션 간에 유지해야 할 수도 있다. 그 경우, `_index` 변수는 앱 상태이다.

There is no clear-cut, universal rule to distinguish whether a particular variable is ephemeral or app state. Sometimes, you’ll have to refactor one into another. For example, you’ll start with some clearly ephemeral state, but as your application grows in features, it might need to be moved to app state.

특정 변수가 일시적인 상태인지 앱 상태인지를 구분하는 명확하고 일반적인 규칙은 없다. 예를 들어, 명백히 일시적인 상태로 시작할 수 있지만 애플리케이션의 기능이 확장됨에 따라 앱 상태로 이동해야 할 수도 있다.

For that reason, take the following diagram with a large grain of salt:

![상태관리 3](https://github.com/changhwan77/changhwan77.github.io/assets/110464205/71d8d0aa-864c-4c08-9046-c0a8ade803a9)

When asked about React’s setState versus Redux’s store, the author of Redux, Dan Abramov, replied:

리액트의 setState와 Redux의 store에 대한 질문에 대해 리덕스의 저자는 다음과 같이 답했다.

> “The rule of thumb is: Do whatever is less awkward.”
> 
> 
> “규칙은 다음과 같다: 가장 부자연스러운 것을 하세요.”
> 

In summary, there are two conceptual types of state in any Flutter app. Ephemeral state can be implemented using `State` and `setState()`, and is often local to a single widget. The rest is your app state. Both types have their place in any Flutter app, and the split between the two depends on your own preference and the complexity of the app.

요약하면, 모든 플러터 앱에는 두 가지 개념적 유형의 상태가 있다. 일시적인 상태는 `State` 와 `setState()` 를 통해서 구현할 수 있고 주로 하나의 위젯에 로컬 상태로 사용된다. 나머지는 앱 상태다. 이 두 유형은 모든 플러터 앱에서 자리를 잡고 있으며, 두 유형의 분할은 개인적인 기호와 앱의 복잡성에 따라 다르다.

여기서 일시적 상태의 **로컬**의 의미는 특정 범위 또는 영역 내에서 적용되는 상태나 데이터를 나타낸다. 하나의 위젯 내에서만 존재하고 해당 위젯과 관련된 상태를 나타낸다.

# **Simple app state management**

선언적 UI 프로그래밍과 일시적 상태와 앱 상태의 차이를 알았다면 간단한 앱 상태 관리에 대해 배울 준비가 되었다고 공식 문서에서 언급하고 있다.

공식 문서에서는 플러터가 처음이고 다른 상태관리 방법을 선택해야 하는 큰 이유가 없다면 `provider` 패키지를 사용하여 설명한다고 적혀져 있다. 왜냐면 `provider` 패키지는 이해하기 쉽고 많은 코드를 사용하지 않고 다른 모든 상태 관리 방법에 적용할 수 있는 개념을 사용하기 때문이라고 한다. 

## **Our example**

아주 간단한 쇼핑 앱으로 설명한다.

![상태관리 4](https://github.com/changhwan77/changhwan77.github.io/assets/110464205/67ea580b-919d-4490-a733-364ee93b5936)

So we have at least 5 subclasses of `Widget`. Many of them need access to state that “belongs” elsewhere. For example, each `MyListItem` needs to be able to add itself to the cart. It might also want to see whether the currently displayed item is already in the cart.

적어도 5개의 `Widget` 하위 클래스가 있는데 그 중 많은 것들이 다른 곳에 “속한” 상태에 접근해야 한다. 예를 들어 `MyListItem` 는 자신을 카트에 추가할 수 있어야 한다. 또한 현재 표시된 항목이 이미 카트에 있는지 확인할 수도 있어야 한다.

This takes us to our first question: where should we put the current state of the cart?

이것이 첫 번째 질문으로 이어진다: 현재 카트의 현재 상태를 어디에 저장하고 접근해야 할까?

## **Lifting state up**

In Flutter, it makes sense to keep the state above the widgets that use it.

플러터에서는 상태를 사용하는 위젯 위에 상태를 유지하는 것이 합리적이다.

Why? In declarative frameworks like Flutter, if you want to change the UI, you have to rebuild it. There is no easy way to have `MyCart.updateWith(somethingNew)`. In other words, it’s hard to imperatively change a widget from outside, by calling a method on it. And even if you could make this work, you would be fighting the framework instead of letting it help you.

플러터와 같은 선언적 프레임워크에서 UI를 변경하려면 UI를 다시 구축해야 한다. `MyCart.updateWith(somethingNew)` 처럼 외부에서 메서드를 호출하여 위젯을 명령형으로 변경하는 것은 쉽지 않다. 그리고 심지어 이 작업을 수행할 수 있다 해도 이것은 프레임워크와 맞서 싸우고 있는 것이다.

즉, 플러터는 명령형 방식으로 `.updateWith(item)` 와 같은 메서드를 호출하여 즉시 UI를 변경하는 것이 아니라 해당 UI를 다시 구성해서 rebuild 하는 것이 일반적이다. 

```dart
// BAD: DO NOT DO THIS
void myTapHandler() {
  var cartWidget = somehowGetMyCartWidget();
  cartWidget.updateWith(item);
}
```

Even if you get the above code to work, you would then have to deal with the following in the `MyCart` widget:

위 코드가 작동한다고 해도, `MyCart` 위젯에서 다음과 같은 문제를 처리해야 한다.

```dart
// BAD: DO NOT DO THIS
Widget build(BuildContext context) {
  return SomeWidget(
    // The initial state of the cart.
  );
}

void updateWith(Item item) {
  // Somehow you need to change the UI from here.
}
```

You would need to take into consideration the current state of the UI and apply the new data to it. It’s hard to avoid bugs this way.

현재 UI의 상태를 고려하고 새 데이터를 적용해야 한다. 이렇게 하면 버그를 마주할 확률이 높다.

이게 무슨 말이냐 하면, 위 코드는 `updateWith` 메서드를 호출해서 ui를 직접 변경하려고 시도하는 것인데, 플러터는 데이터가 변경될 시 UI를 그리는 위젯 자체를 새로 만들 화면을 그리도록 하므로 이렇게 Imperative 하게 상태를 관리하는 것은 공식 문서에서 사용하지 말라고 한다. 

In Flutter, you construct a new widget every time its contents change. Instead of `MyCart.updateWith(somethingNew)` (a method call) you use `MyCart(contents)` (a constructor). Because you can only construct new widgets in the build methods of their parents, if you want to change `contents`, it needs to live in `MyCart`’s parent or above.

플러터에서는, 위젯의 설정 정보가 변경될 때마다 새 위젯을 구성한다. `MyCart.updateWith(somethingNew)` 처럼 메서드를 호출하기 보다는 `MyCart(contents)` (생성자)를 사용하는 게 좋다. 부모 위젯의 `build( )` 메서드 내에서 새 위젯을 구성할 수 있으므로, `contents`를 변경하길 원한다면, `MyCart`의 부모 또는 그 이상에 있어야 한다.

이 말은 `MyCart`라는 위젯을 자식으로 두는 부모 위젯이 있는데 이 부모 위젯의 `build( )` 내에서 

`MyCart(contents)` 가 가진 `contents` (상태, 여기서는 쇼핑 카트에 담긴 아이템들)가 변경되면 이를 감지하여 부모 위젯인 `App` 이 다시 새 `MyCart` 를 생성하며 새로운 `contents` 를 담은 UI를 그리게 된다. 

```dart
// GOOD
void myTapHandler(BuildContext context) {
  var cartModel = somehowGetMyCartModel(context);
  cartModel.add(item);
}
```

Now `MyCart` has only one code path for building any version of the UI.

이제 `MyCart` 에는 UI의 모든 버전을 빌드하는 하나의 코드 경로만 있다.

```dart
// GOOD
Widget build(BuildContext context) {
  var cartModel = somehowGetMyCartModel(context);
  return SomeWidget(
    // Just construct the UI once, using the current state of the cart.
    // ···
  );
}
```

In our example, `contents` needs to live in `MyApp`. Whenever it changes, it rebuilds `MyCart` from above (more on that later). Because of this, `MyCart` doesn’t need to worry about lifecycle—it just declares what to show for any given `contents`. When that changes, the old `MyCart` widget disappears and is completely replaced by the new one.

이 예제에서 `contents` 는 `MyApp` 위젯에 있어야 한다. 데이터가 변경될 때마다 `MyCart` 위젯을 재구축한다. 이로 인해 `MyCart` 는 수명주기를 걱정할 필요가 없으며, 그냥 `contents`에 따라 어떤 내용을 보여줄 지만 선언하면 된다. 그 내용이 변경되면 이전 `MyCart`  위젯은 완전히 새로운 것으로 교체된다. 

여기서 lifecycle이란 앱이 시작되고, 실행, 종료될 때까지 거치는 단계 및 이벤트의 순서를 나타낸다. 쇼핑 앱에 유저가 들어와서 물건들을 보고 맘에 들면 장바구니에 물건을 추가하게 되는데, 물건을 추가하게 될 시, `MyApp` 이라는 위젯에서 새로운 데이터를 감지하고 `MyCart` 에 해당 `contents` 를 전달하여 새로운 `MyCart`  위젯을 생성하여 새로 추가한 `contents` 를 담은 UI를 그리게 된다. `MyCart`  위젯은 그때까지의 라이프사이클에 대해 걱정할 필요 없이 현재 상태를 나타내기만 하면 된다는 의미이다.

![상태관리 5](https://github.com/changhwan77/changhwan77.github.io/assets/110464205/a39d315a-8093-4288-b749-1a5a304ce4de)

This is what we mean when we say that widgets are immutable. They don’t change—they get replaced.

이것이 위젯이 불변(immutable)이라고 말하는 것이다. 그들은 변경되지 않고 대체된다!

Now that we know where to put the state of the cart, let’s see how to access it.

이제 카트의 상태를 어디에 놓아야 하는지 알았으니, 이를 어떻게 접근(access)할 수 있는지 알아보자. 

## **Accessing the state**

When a user clicks on one of the items in the catalog, it’s added to the cart. But since the cart lives above `MyListItem`, how do we do that?

A simple option is to provide a callback that `MyListItem` can call when it is clicked. Dart’s functions are first class objects, so you can pass them around any way you want. So, inside `MyCatalog` you can define the following:

여기서 일급 객체(First-class object)란 프로그래밍 언어에서 특별한 권한 없이 변수에 할당하거나 함수의 매개변수로 전달하거나 함수의 반환 값으로 사용할 수 있는 개체를 가리킨다. 

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

This works okay, but for an app state that you need to modify from many different places, you’d have to pass around a lot of callbacks—which gets old pretty quickly.

위와 같이 콜백을 사용하면 MyListItem이 선택된 상품을 장바구니에 추가할 수 있다. 그러나 이 방식은 앱의 여러 위치에서 상태를 변경해야 할 때 콜백을 전달하는 것이 번거롭고 복잡해질 수 있다.

Fortunately, Flutter has mechanisms for widgets to provide data and services to their descendants (in other words, not just their children, but any widgets below them). As you would expect from Flutter, where *Everything is a Widget™*, these mechanisms are just special kinds of widgets—`InheritedWidget`, `InheritedNotifier`, `InheritedModel`, and more. We won’t be covering those here, because they are a bit low-level for what we’re trying to do.

Instead, we are going to use a package that works with the low-level widgets but is simple to use. It’s called `provider`.

Before working with `provider`, don’t forget to add the dependency on it to your `pubspec.yaml`.

To add the `provider` package as a dependency, run `flutter pub add`:

```dart
flutter pub add provider
```

Now you can `import 'package:provider/provider.dart';` and start building.

With `provider`, you don’t need to worry about callbacks or `InheritedWidgets`. But you do need to understand 3 concepts:

- ChangeNotifier
- ChangeNotifierProvider
- Consumer

## **ChangeNotifier**

`ChangeNotifier` is a simple class included in the Flutter SDK which provides change notification to its listeners. In other words, if something is a `ChangeNotifier`, you can subscribe to its changes. (It is a form of Observable, for those familiar with the term.)

`ChangeNotifier` 는 Flutter SDK에 포함된 간단한 클래스로, 리스너에게 변경 사항을 알려주는 역할을 한다. 무언가가 `ChangeNotifier` 인 경우, 그 변경사항에 대한 구독이 가능하다.

In `provider`, `ChangeNotifier` is one way to encapsulate your application state. For very simple apps, you get by with a single `ChangeNotifier`. In complex ones, you’ll have several models, and therefore several `ChangeNotifiers`. (You don’t need to use `ChangeNotifier` with `provider` at all, but it’s an easy class to work with.)

`provider` 에서 `ChangeNotifier` 는 앱 상태를 캡슐화하는 한 가지 방법이다. 매우 간단한 앱의 경우, 하나의 `ChangeNotifier` 로 충분할 수 있다. 복잡한 앱의 경우, 여러 모델이 필요하며 여러 `ChangeNotifiers` 가 필요할 수 있다.

In our shopping app example, we want to manage the state of the cart in a `ChangeNotifier`. We create a new class that extends it, like so:

쇼핑 앱 예제에서는,장바구니의 데이터를 `ChangeNotifier` 로 관리하고자 한다. 다음과 같이 이를 `ChangeNotifier`를 상속받는 새로운 클래스를 만든다.

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

The only code that is specific to `ChangeNotifier` is the call to `notifyListeners()`. Call this method any time the model changes in a way that might change your app’s UI. Everything else in `CartModel` is the model itself and its business logic.

`ChangeNotifier` 와 관련한 유일한 코드는 `notifyListeners()` 를 호출하는 부분이다. 이 메서드를 호출하면 모델이 변경되어 앱의 ui를 변경할 수 있는 방식으로 변경된 경우에 호출되어야 한다. 다른 모든 부분, 즉 `CartModel` 내의 나머지 부분은 모델 자체와 해당 비즈니스 로직에 관한 것이다.

`ChangeNotifier` is part of `flutter:foundation` and doesn’t depend on any higher-level classes in Flutter. It’s easily testable (you don’t even need to use [widget testing](https://docs.flutter.dev/testing/overview#widget-tests) for it). For example, here’s a simple unit test of `CartModel`:

```dart
test('adding item increases total cost', () {
  final cart = CartModel();
  final startingPrice = cart.totalPrice;
  var i = 0;
  cart.addListener(() {
    expect(cart.totalPrice, greaterThan(startingPrice));
    i++;
  });
  cart.add(Item('Dash'));
  expect(i, 1);
});
```

## **ChangeNotifierProvider**

`ChangeNotifierProvider` is the widget that provides an instance of a `ChangeNotifier` to its descendants. It comes from the `provider` package.

`ChangeNotifierProvider` 는 `ChangeNotifier` 의 인스턴스를 해당 자손 위젯에 제공하는 위젯임. 이 위젯은 `provider` 패키지에서 제공됨. 

We already know where to put `ChangeNotifierProvider`: above the widgets that need to access it. In the case of `CartModel`, that means somewhere above both `MyCart` and `MyCatalog`.

`CartModel` 의 경우, `MyCart` 와 `MyCatalog` 두 위젯 모두에서 접근해야 하므로 해당 위젯들 보다 상위에 있어야 함.  즉, 데이터가 필요한 위젯 보다 상위에 `ChangeNotifierProvider` 을 두어야 함.

You don’t want to place `ChangeNotifierProvider` higher than necessary (because you don’t want to pollute the scope). But in our case, the only widget that is on top of both `MyCart` and `MyCatalog` is `MyApp`.

여기서 중요한 점은 `ChangeNotifierProvider` 를 필요한 것보다 더 상위에 놓지 않는 것임. 따라서 

`MyCart` 와 `MyCatalog` 두 위젯 모두 위에 있는 `MyApp` 임.

즉, 데이터를 사용하려는 위젯의 상위에 `ChangeNotifierProvider` 를 제공하면 `ChangeNotifier` 를 상속받은 클래스의 데이터들에 접근이 가능함. 

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

Note that we’re defining a builder that creates a new instance of `CartModel`. `ChangeNotifierProvider` is smart enough *not* to rebuild `CartModel` unless absolutely necessary. It also automatically calls `dispose()` on `CartModel` when the instance is no longer needed.

If you want to provide more than one class, you can use `MultiProvider`:

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

## Consumer

Now that `CartModel` is provided to widgets in our app through the `ChangeNotifierProvider` declaration at the top, we can start using it.

This is done through the `Consumer` widget.

이제 `CartModel` 은 `ChangeNotifierProvider` 선언을 통해 앱의 위젯에 제공되었으므로 이를 사용할 수 있음. 

이것은 `Consumer` 위젯을 통해 수행됨.

```dart
return Consumer<CartModel>(
  builder: (context, cart, child) {
    return Text('Total price: ${cart.totalPrice}');
  },
);
```

We must specify the type of the model that we want to access. In this case, we want `CartModel`, so we write `Consumer<CartModel>`. If you don’t specify the generic (`<CartModel>`), the `provider` package won’t be able to help you. `provider` is based on types, and without the type, it doesn’t know what you want.

데이터를 사용하려는 모델의 타입을 명시해야 함. 이 경우 `CartModel` 를 사용해야 하므로 `Consumer<CartModel>` 를 작성함. 제네릭(`<CartModel>`)을 지정하지 않으면, `provider` 패키지가 도움을 줄 수 없음. `provider` 는 타입을 기반으로 하며, 타입이 없으면 원하는 것을 알 수 없음.

The only required argument of the `Consumer` widget is the builder. Builder is a function that is called whenever the `ChangeNotifier` changes. (In other words, when you call `notifyListeners()` in your model, all the builder methods of all the corresponding `Consumer` widgets are called.)

`Consumer` 의 필수 인자는 `builder` 임. 빌더는 `ChangeNotifier` 가 변경될 때마다 호출되는 함수임. (다시 말해, 모델에서 `notifyListeners()` 를 호출할 때 해당 `Consumer` 위젯의 모든 빌더 메서드가 호출됨.)

즉, `Consumer` 위젯을 통해 사용하려는 모델의 데이터에 접근할 수 있고 데이터가 변경될 때마다 이 `Consumer` 위젯을 사용한 모든 위젯의 `builder` 가 호출됨으로써 데이터가 변경된 ui를 반영하여 그리게 됨.

The builder is called with three arguments. The first one is `context`, which you also get in every build method.

빌더 함수에는 세 인자가 함께 호출됨. 첫 번째는 `context` 이며, 이것은 모든 빌드 메서드에서 사용할 수 있음. 이 인자로 인해 `Consumer` 위젯을 사용하는 곳을 청취하다가 데이터가 변경될 시 이를 알림을 받고 ui를 재빌드할 수 있음.

The second argument of the builder function is the instance of the `ChangeNotifier`. It’s what we were asking for in the first place. You can use the data in the model to define what the UI should look like at any given point.

두 번째 인자는 `ChangeNotifier` 의 인스턴스임. 모델의 데이터를 통해 어떤 시점에서 ui가 어떻게 보여야 하는지 정의할 수 있음. 이 부분은 위에서 `ChangeNotifier` 를 상속하여 설계한 `CartModel` 의 인스턴스를 뜻함. 이 모델에서 제공하는 데이터(예:totalPrice)를 사용하여 UI를 생성하는 데 사용할 수 있음. 

The third argument is `child`, which is there for optimization. If you have a large widget subtree under your `Consumer` that *doesn’t* change when the model changes, you can construct it once and get it through the builder.

세 번째 인자는 최적화를 위한 `child` 임. 모델이 변경되어도 변경되지 않는 큰 위젯 하위 트리를 한 번만 생성하고 빌더를 통해 얻을 수 있음. 

예를 들어, `Consumer` 내에 있는 하위 트리가 변경되지 않는 경우 다음과 같이 최적화를 할 수 있음. 

```dart
return Consumer<CartModel>(
  builder: (context, cart, child) => Stack(
    children: [
      // Use SomeExpensiveWidget here, without rebuilding every time.
      if (child != null) child,
      Text('Total price: ${cart.totalPrice}'),
    ],
  ),
  // Build the expensive widget here.
  child: const SomeExpensiveWidget(),
);
```

It is best practice to put your `Consumer` widgets as deep in the tree as possible. You don’t want to rebuild large portions of the UI just because some detail somewhere changed.

`Consumer` 위젯을 가능한 깊게 배치하는 것이 best practice임. 어딘가가 변경되었을 때 큰 부분의 UI를 다시 빌드하지 않으려면 이렇게 하는 게 좋음.

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

Instead:

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

### Provider.of

Sometimes, you don’t really need the *data* in the model to change the UI but you still need to access it. For example, a `ClearCart` button wants to allow the user to remove everything from the cart. It doesn’t need to display the contents of the cart, it just needs to call the `clear()` method.

가끔 모델의 데이터가 UI를 변경할 필요는 없지만 액세스해야 할 때가 있음. 예를 들어, `ClearCart` 버튼은 사용자가 장바구니에서 모든 항목을 제거할 수 있도록 허용하려면 해당 카트의 데이터를 보여줄 필요는 없지만 `clear()` 메서드를 호출해야 함.

We could use `Consumer<CartModel>` for this, but that would be wasteful. We’d be asking the framework to rebuild a widget that doesn’t need to be rebuilt.

이 때 `Consumer<CartModel>`를 사용할 수는 있지만, 낭비일 수 있음. UI를 다시 빌드할 필요가 없는 위젯을 다시 빌드하도록 프레임워크에 요청하는 것이기 때문임.

For this use case, we can use `Provider.of`, with the `listen` parameter set to `false`.

이러한 사용 사례에 대해 `listen` 매개변수를 `false` 로 설정하여, `Provider.of` 를 사용할 수 있음.

```dart
Provider.of<CartModel>(context, listen: false).removeAll();
```

Using the above line in a build method won’t cause this widget to rebuild when `notifyListeners` is called.****

위와 같은 코드를 빌드 메서드에서 사용하는 경우, `notifyListeners` 가 호출될 때 이 위젯을 다시 빌드하지 않음.


