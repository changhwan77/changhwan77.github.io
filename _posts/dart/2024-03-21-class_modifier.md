---
layout: single
title: 인터페이스와 추상 클래스 이해하기
categories:
  - Dart
tags:
  - [ OOP, Interface, abstract ]
toc: true
date: 2024-03-21
---

클래스는 해당 객체의 type(유형)을 나타내는 것이고 두 type 사이의 관계를 표현하는 방법있다. 

### 상속(Inheritance)

가장 먼저 클래스를 상속하는 것이다. 즉, 강아지라는 클래스와 동물이라는 클래스가 있을 때 dog 클래스가 extends 키워드를 사용해 Animal 클래스를 
상속하게 되는데 이는 Animal 클래스의 내부 구현 사항을 모두 사용할 수 있음을 의미한다. 

### Interface

클래스와 달리 Interface는 생성자, 속성, 메서드의 body 모두 없고 메서드의 시그니처만 선언한 것이다. 즉, 이 객체가 하는 행동들은 메서드의 시그니처
로 제공하고 세부 구현 사항은 제공하지 않는 것이다. 

```dart
interface Animal {
  void makeSound();
}
```

즉, makeSound라는 메서드의 시그니처만 선언해놓고 이는 일종의 계약으로 Animal를 구현하는 클래스에서 해당 계약을 이행해야 한다. 즉, makeSound라는
메서드의 세부 구현을 완성해서 사용해야 한다.

예를 들어 Cat이라는 클래스가 Animal 이라는 인터페이스를 구현한다면 다음과 같이 사용할 수 있다.

```dart
class Cat implements Animal { 
  void makeSound() { 
    print('Meau');
  };
}
```

### Abstract 

Dart 공식문서에 따르면 전체 interface의 전체 또는 구체적인 구현 사항이 필요 없는 class를 정의할 때 abstract 클래스 modifier를 사용한다고 한다.
추상 클래스로 객체를 생성할 수 없고 다음과 같이 사용할 수 있다.

```dart
abstract class Vehicle{ 
  void moveForward(int meters);
}
```

```dart 
// 구현할 수 있고
class Car extends Vehicle{
  @override
  void moveForward(int meters){
  ...
  }
}

```dart 
// 상속할 수 있다.
class Car extends Vehicle{
  int passsengrs =4;
  }
}
```

### interface와 추상 클래스의 차이점

아직 명확히 파악은 못했지만, 
- interface: 메서드 시그니처(해당 클래스를 이행시 어떤 행동을 해야하는 지에 대한 설명)만 있고 나머지는 선언이 불가능하고 다중 상속이 가능하다. 
- abstract: 일반 클래스처럼 세부 구현 사항을 선언하고 상속할 수 있돼, 다중 상속이 불가능하다. 추상 클래스는 여러 클래스의 공통적인 기능을 공유하는 데 사용한다.
