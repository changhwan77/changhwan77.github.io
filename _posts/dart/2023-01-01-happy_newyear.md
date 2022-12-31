---
layout: single
title: 햎피 뉴이어 
categories:
  - Dart
tags:
  - [Flutter]
toc: true
date: 2023-01-01
---

### 적응형(adaptive)와 반응형(responsive) 앱의 차이점

**Responsive**

사용자의 화면 크기에 맞게 레이아웃을 조정하는 것. 즉, 사용자가 창 크기를 조절하거나 장치를 세로 또는 가로로 방향을 변경할 때 UI를 재배치하는 것을 말함.

**Adaptive**

사용자가 입력하는 걸 처리하는 것 (사용자 친밀도!) 

### 반응형(responsive) 앱 만드는 법

플러터는 반응형 디자인을 적용한 앱을 만드는데 2가지의 기본적인 접근법을 사용함. 

1. **Use the `[LayoutBuilder](https://api.flutter.dev/flutter/widgets/LayoutBuilder-class.html)` class**
2. **Use the `[MediaQuery.of()](https://api.flutter.dev/flutter/widgets/MediaQuery/of.html)` method in your build functions**

# 적응형(adaptive) 앱 만드는 법

## 폼 팩터와 스크린 크기 고려

![폼팩터](https://user-images.githubusercontent.com/110464205/210145657-51d231ce-a85b-437e-aa7f-e20ce4558bf5.jpg)


### 레이아웃 위젯들

- **Single child**
    - **Align**
    - **AspectRatio**
    - **ConstrainedBox**
    - **CustomSingleChildLayout**
    - **Expanded and Flexible**
    - **FractionallySizedBox**
    - **LayoutBuilder**
    - **SingleChildScrollView**
    
- **Multichild**
    - Column, Row and Flex
    - CustomMultiChildLayout
    - Flow
    - ListView, GridView and CustomScrollView
    - Stack
    - Table
    - Wrap

### 시각적 밀도 (Visual density)

사용자가 원함에 따라 전체 어플리케이션의 밀도를 조정할 수 있음. ex) 버튼을 더 크게 만드는 등 

### 화면 크기에 따라 레이아웃 크기 설정 (Contextual layout)

- **Screen-based breakpoints**

![화면 크기에 따라 수평_수직으로 달라지는 위젯](https://user-images.githubusercontent.com/110464205/210146331-c933c2cb-623e-465d-8e88-107afc42282f.png)
![달라지는 위젯 2](https://user-images.githubusercontent.com/110464205/210146371-4bb91fac-665e-418b-907d-08996042fd04.png)

    - 데스크탑, 태블릿, 등 스크린 크기의 기준을 임의로 설정해서 기준을 벗어날 시 위젯들을 수평 또는 수직으로 변환하도록 함.

<iframe src="https://dartpad.dev/embed-flutter.html?id=337bec81c0b31f91706b14749cd1d6d9" style="width:120%; height:300px"></iframe>
    
- **Use LayoutBuilder for extra flexibility**

폼팩터들의 전체화면 말고 스크린 내 더 세밀한 부분의 레이아웃을 조정해야 한다면 플러터에선 `LayoutBuilder` 로 조정함. 

- **Device segmentation**
    - 스크린 사이즈 조정이 아니라 플랫폼에 따라 설정 가능

```dart
bool get isMobileDevice => !kIsWeb && (Platform.isIOS || Platform.isAndroid);
bool get isDesktopDevice =>
    !kIsWeb && (Platform.isMacOS || Platform.isWindows || Platform.isLinux);
bool get isMobileDeviceOrWeb => kIsWeb || isMobileDevice;
bool get isDesktopDeviceOrWeb => kIsWeb || isDesktopDevice;
```

### **Single source of truth for styling**

화면을 설계할 때 들어가는 폰트의 스타일이나 padding 등은 디자인 원칙상 일관성을 유지해야 함. 이를 더 쉽게 유지보수 하기 위해서 관련 클래스를 생성하는 습관 들이기. 

**코드 예시**

```dart
class Insets {
  static const double xsmall = 3;
  static const double small = 4;
  static const double medium = 5;
  static const double large = 10;
  static const double extraLarge = 20;
  // etc
}

class Fonts {
  static const String raleway = 'Raleway';
  // etc
}

class TextStyles {
  static const TextStyle raleway = const TextStyle(
    fontFamily: Fonts.raleway,
  );
  static TextStyle buttonText1 =
      TextStyle(fontWeight: FontWeight.bold, fontSize: 14);
  static TextStyle buttonText2 =
      TextStyle(fontWeight: FontWeight.normal, fontSize: 11);
  static TextStyle h1 = TextStyle(fontWeight: FontWeight.bold, fontSize: 22);
  static TextStyle h2 = TextStyle(fontWeight: FontWeight.bold, fontSize: 16);
  static late TextStyle body1 = raleway.copyWith(color: Color(0xFF42A5F5));
  // etc
}
```

이렇게 내가 사용하려는 스타일링에 따라 클래스를 생성하여 내부에 constants들을 생성해놓고 앞으로 위젯을 생성할 때 사용.

**코드 예시**

```dart
return Padding(
  padding: EdgeInsets.all(Insets.small),
  child: Text('Hello!', style: TextStyles.body1),
);
```

위에서는 Insets의 프로퍼티 small 이랑 TextStyles의 프로퍼티인 body1 사용.

이런식으로 동일하게 공유되는 디자인 시스템을 설계하여 사용하면 우리가 만드는 View들이 보기가 더 좋아지고 일관성이 높아짐. 

일반적으로 쓰이는 디자인 시스템 카테고리들 

- Animation timings
- Sizes and breakpoints
- Insets and paddings
- Corner radius
- Shadows
- Strokes
- Font families, sizes, and styles

**앞으로 이렇게 한 곳에 모아놓고 쓰기 !** 

### 각 폼 팩터의 강점에 맞게 설계하기 **( Design to the strengths of each form factor )**

화면 크기 외에도 다양한 폼 팩터의 고유한 장단점을 고려하는 데 시간을 투자해야 함. 

예를 들어, 모바일 기기는 휴대용에 카메라가 있지만 아주 디테일한 창의적인 일을 하는데는 적합하지 않음.  그보단 콘텐츠를 캡쳐하는데 집중. 반면에 태블릿이나 데스크톱 UI는 콘텐츠를 구성하거나 조작하는데 집중.

여기서 핵심은 각 플랫폼이 가장 잘하는 것이 무엇인지 생각하고 활용할 수 있는 고유한 기능이 있는지 확인하는 것.

### 빠른 테스트를 위한 데스크탑 빌드 대상을 사용하기

테스트할 때는 데스크탑으로 실행해서 실험하는 게 젤 효과적인 방법쓰 ~ 

### 터치를 먼저 해결하기 (Solve touch first)

가장 중요한 점은 유저가 어떤 앱을 사용할 때 무엇을 기대할지 고려하는 것 ! 

## Input (사용자 입력 방식 지원 )

### **Scroll wheel**

보통 `ScrollView` 나 `ListView` 위젯이 기본적으로 스크롤 휠을 제공함. 

### **Tab traversal and focus interactions**

키보드 사용자는 탭 키를 사용해서 어플리케이션을 빠르게 탐색하길 원함. 

관련 위젯으로는 button 이랑 textfield 위젯 

### **Controlling traversal order**

### **Keyboard accelerators**

사용자들은 다음과 같은 키보드 단축키 (shortcuts)을 원함 ! 

`Delete` key  or `Control+F`  
유저들이 기대하는 이런 것들을 고려해야 됌 ! 

# 사용자가 각 플랫폼에 기대하는 관례들

- **인지적 부하를 감소시키기**
    - 내가 만든 앱을 사용하는 게 직관적이고, 덜 생각하고 생산성을 향상시켜야 함.
- **신뢰감 형성**
    - 사용자가 자신의 기대에 미치지 못하는 앱을 사용할 경우 만족하지 못함. 따라서 친숙하게 느껴지는 UI를 고려해야 됌.


## 일반적으로 고려되는 규칙들

### 스크롤 막대 모양 및 동작 (**Scrollbar appearance and behavior )**

- 모바일 유저는 스크롤을 할 때만 작게 스크롤바가 나타나길 원하지만
- 데스크톱 유저는 일반적으로 그들이 클릭하거나 드래그할 수 있는 큼직한 스크롤 바를 원함.
- 플러터에서는 Scrollbar 위젯 지원 중

### Selectable text

<iframe src="https://dartpad.dev/embed-flutter.html?id=51c86c4f7c01be7ada883f9a4bb8e615" style="width:120%; height:300px"></iframe>

### ToolTip

<iframe src="https://dartpad.dev/embed-flutter.html?id=61a5ad666a2ddae14c0b1b70a619414c" style="width:120%; height:300px"></iframe>
  
### Horizontal button order

<iframe src="https://dartpad.dev/embed-flutter.html?id=7b7582020a4aa72b81eba4b4f2c41597" style="width:120%; height:300px"></iframe>

## React 를 시작하다

사실 나는 프로젝트 기반 프로그래밍 교육 플랫폼을 만들고 싶은데 .... 아무리 생각해도 웹이 제일 적격인 것 같다.
오늘 이 공식문서 스터디를 하면서도 웹이 창의적인 활동에 적합하다고 (당연히 모바일은 간단한거 위주) 라고 해서 더 생각이 굳어진 것 같다. 

따라서 ..... 

![image](https://user-images.githubusercontent.com/110464205/210148558-71f98458-2afb-4231-a70b-cc7f1ad56eb3.png)

바로 뭐다 ? 상상을 현실로 이루기 위해 사는 나 ㅎㅎ 

7개월 과정 2주만에 끝낼 예정 ㅋ ㅋ 바로 시작 빠빠이 

그리고 해피유희열 !!!!! 
