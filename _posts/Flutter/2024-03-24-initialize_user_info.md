---
layout: single
title: 클린 아키텍쳐 적용 후 UiState 초기화 문제 기록
toc: true
date: 2024-03-24
categories:
  - Flutter
tag: [Flutter, Provider, Future]
---

## 문제 상황
기숙사 식단 홈화면 진입 시 서버로부터 유저 정보를 받아와서 UiState로 가공해(초기화하고) 해당 데이터를 필요한 UI Element에 뿌리는 작업을 진행했다.

![image](https://github.com/changhwan77/changhwan77.github.io/assets/110464205/414e087a-19be-4f63-a8e2-78e306e1455b)

즉, 이 화면에서 유저의 정보를 초기화하고 이 정보에 기반해 AppBar의 Title을 설정해주는 작업이었다. 
<br>
![image](https://github.com/changhwan77/changhwan77.github.io/assets/110464205/4d60909c-90f6-463c-b5f9-b4f358f5ef3d)
<br>
본래는 userViewModel이라는 객체를 late 키워드를 통해 선언하고 initState를 통해 초기화가 완료되면 notifyListner를 통해 화면이 갱신되도록 했다.
하지만 이렇게 설정시 화면 빌드가 두 번이 되어서 정확한 건 아니지만 성능에 오류가 있을 것 같다는 느낌이 들어 한 번만 빌드되도록 하고 싶어 해결을 시작했다. 

### AppBar에 FutureBuilder 사용해 초기화하기

AppBar의 title 프로퍼티에 FutureBuilder를 사용해 User객체 초기화가 끝나면 uiState를 초기화하고 이를 사용해 Text를 빌드하려고 했으나 
ref 객체가 먼저 있어서 null 값을 참조해 오류가 발생했다. 

그리고 이렇게 설정하면 해당 화면이 빌드될 떄마다 유저 객체를 서버에서 불러온다는 점에서 비효율적이라는 생각이 들었다. 

## 해결 방법
따라서 홈화면이 1번만 빌드되기 위해선 첫 진입 시 유저 객체를 한 번만 초기화하고 후에는 유저가 기숙사를 변경할 때마다 해당 Provider가 호출되므로 
다음과 같이 설정해줬다.

![image](https://github.com/changhwan77/changhwan77.github.io/assets/110464205/2d052f63-008d-43d7-8b1c-371332050952)
<br>
![image](https://github.com/changhwan77/changhwan77.github.io/assets/110464205/f1b3f341-12ea-4d0c-aacc-3346c872b930)
<br>
![image](https://github.com/changhwan77/changhwan77.github.io/assets/110464205/f5172236-9d8d-421e-a974-7be43a01491a)
<br>
즉, 앱의 진입점에서 User객체를 초기화하고 UiState를 setter를 통해 초기화해준다. 그리고 앱바가 포함된 build메서드에서 이를 watch하여 
초기화된 uiState를 참조해 앱바의 타이틀을 표시한다. 

이렇게 했더니 홈화면에서 build했을 때 uiState의 값을 참조할 수 있었고 빌드가 2번되는 문제를 해결할 수 있었다.
