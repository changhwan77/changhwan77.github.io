---
layout: single
title: 플러터 스터디 1
categories:
  - Dart
tags:
  - [Flutter]
toc: true
date: 2022-11-18
---

### AspectRatio class

- AspectRatio란 이미지나 화면의 너비와 높이의 비율
    - child의 화면 비율을 조절할 수 있는 위젯
    
    ```dart
    AspectRatio(
            aspectRatio: 16 / 9,
            child: Container(
              color: Colors.green,
            ),
          ),
    ```
    
    - child인 컨테이너가 ui에 보여지는 비율을 조절하는 데
    - aspectRatio라는 property에 16 / 9 처럼 비율을 조절할 수 있음.
    
    [AspectRatio class - Dart API](https://api.flutter.dev/flutter/widgets/AspectRatio-class.html)
    

### ListView.builder

- 내가 원하는 만큼 스크롤 가능한 위젯들 생성 가능.
    - 이 생성자는 children에 들어갈 위젯 수가 많거나 갯수가 무한할 때 쓰기 적합함.
- Property
    - itemCount
        - 에 내가 ListView에 생성할 위젯 수를 설정할 수 있음.
        - 보통 이 property의 값은 내가 만들어놓은 리스트 등의 이름.length를 사용함. 이유는 리스트 내 값들이 많을 수록 사람보다 컴퓨터가 그 수를 계산하는 게 빠르고 정확해서 그런 듯.
    - itemBuilder
        - null이 아닌 위젯만 return 가능
        - 형식은 익명 함수 ( ) { }로
            - 함수 바디 안에 내가 생성하고 싶은 위젯을 return
            - 이 때 인덱스 순으로 순환되기 때문에 밑의colorCodes[index] 처럼 코드 작성.
            - 의문점. 이 때 만약 데이터를 우리가 만드는 게 아니라 외부에서 받아오는 경우,,, ? 어떻게 로직을 짤 수 있을지 !
 
<iframe src="https://dartpad.dev/embed-flutter.html?id=cf8749fdab5976cc2a11719b6e4615b6" style="width:100%; height:500px"></iframe>

[ListView.builder constructor - Dart API](https://api.flutter.dev/flutter/widgets/ListView/ListView.builder.html)

### GestureDetector

- 제스처를 감지하는 위젯
    - 클릭, 더블 클릭, 긴 클릭, 드래그, 스와이프 등 다양하게 가능

<iframe src="https://dartpad.dev/embed-flutter.html?id=d70d2487c6ab42cba8b3194205801786" style="width:100%; height:500px"></iframe>

- onTap 함수 바디 안에 setState를 설정. 
    - 여기서 setState란 한 이 페이지의 코드 내에서 더럽혀진, 즉 상태가 바뀔 수 있는 값을 설정하고 내가 변경하고 싶은 대로 가능. 
