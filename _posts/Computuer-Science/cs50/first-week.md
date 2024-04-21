---
layout: single
title: (CS50) - Dart와 비교하며 C언어 알아보기
toc: true
date: 2024-04-21
categories:
  - CS50
tag: [C언어]
---

회사에서 개발자를 하려면 C언어는 무조건 알아야 된다고 해서 주말 동안 시간이 남으면 C언어에 대한 기초를 공부해보면 좋을 것 같다고 하셨고 원래 컴개론을 공부하면서 C언어가 나오기도 해서 이번 주말을 활용해 스터디를 진행했다. 
이때까지 배운 dart언어와 비교하면서 스터디를 하니 이해가 수월했기도 하고 조금씩 내 예상과 어긋나는 부분도 더러 있었다.

### Hello World!
무슨 언어를 배우든 "hello world!" 부터 출력하는 것으로 시작하는데, 마찬가지다.<br>

```c
#include <stdio.h>

int main(void) {
  printf("hello 창환!");
}
```
<br>

먼저 생소했던 부분은 #include syntax인데, 이는 dart의 import와 같은 역할을 하는 것이었다. 
즉, #include로 선언된 부분은 컴파일러에게 클라우드 환경에 <stdio.h>라는 파일이 있는데 이를 들여다보라는 말을 전달하는 것이다. 

```dart
import 'dart:convert';

void main( ) {
  print("hello 창환!");
}
```

그리고 특이한 점은 함수의 인자에 void를 선언해주는 것이었는데, 아직 이해는 하지 못했다. 그리고 함수의 return type이 int라는 점도.

두 번째로는 콘솔에 출력해보려면 printf 함수를 사용해야 하는데 이를 사용하려면 stdio.h 라는 파일에 있기 때문에 이를 들여다보라고 #include 를 통해 알려주는 것이다.

## 파일을 실행하려면? 

### clang 컴파일러 사용하기
dart에서는 파일을 실행하려면 그냥 소스 코드를 작성하고 run을 눌러주기만 하면 알아서 파일이 실행됐지만 C언어는 달랐다. 

바로 clang 이라는 컴파일러에게 컴파일을 하여 Binary값으로 이루어진 오브젝트 코드를 생성해달라고 요청을 해야 한다.<br>
<img width="567" alt="image" src="https://github.com/changhwan77/changhwan77.github.io/assets/110464205/38303253-b3df-4bf3-86be-3a954e13966e"><br>
터미널에서 프롬프트를 통해 clang hello.c -lcs50 이라는 명령어를 사용하면 왼쪽 탐색기에서 a.out이라는 오브젝트 파일을 컴파일러가 생성해준다. 

이 때 a.out이라는 파일명이 생성하다면 명령행 인자인 -o를 사용해 파일 이름을 지정할 수 있고 -l의 경우에는 컴파일러에게 cs50이라는 파일을 오브젝트 코드로 변환해 내 파일과 합쳐달라고 알려주는 것이다. 

그럼 예를 들어 example이라는 파일명으로 오브젝트 파일을 생성하고 싶다면 다음과 같이 명령어를 사용하면 된다.
clang -o example hello.c -lcs50 

### 파일 실행해보기
이렇게 컴파일된 오브젝트 파일이 생성됐다면 해당 파일의 path를 입력하면 파일이 실행된다.<br>
<img width="262" alt="image" src="https://github.com/changhwan77/changhwan77.github.io/assets/110464205/65c77f27-ece2-4559-9ef7-c4576de9c16c"><br>

## 문자열(String)
dart와는 달리, 문자열을 나타내는 class명이 string이었다. 

dart에서는 다음과 같이 문자열을 변수에 할당하고 그 변수를 문자열 내에서 binding하여 사용하고 싶다면 기호 $(dollar sign)을 사용한다.<br>

```dart
import 'dart:convert';

void main( ) {
  String name = "창환";
  print("hello $name!");
}
```

<br>
이와는 달리 C언의 경우에는 형식 지정자라는 syntax를 사용하는데 예시는 다음과 같다. 

```c
#include <stdio.h>

int main(void) {
  string name = "창환";
  printf("hello %s\n", name); // hello 창환
}
```
즉 할당하려는 자리에 %(percent sign)과 s(string)을 합쳐서 %s로 기입한다. 그리고 문자열이 끝났음을 알리는 "(double quote)뒤에 ,(comma) 바로 옆에 할당하려는 문자열을 담은 변수를 선언한다.

<img width="269" alt="image" src="https://github.com/changhwan77/changhwan77.github.io/assets/110464205/59f98495-c4c8-48d0-aa2c-d601980374e2"><br>

**그럼 변수 두 개 이상을 할당하고 싶으면 어떨까?**

### C언어
```c
#include <stdio.h>

int main(void) {
    string name = "창환";
    string job = "Software Engineer";
    int pay = 3000000;
    printf("Hello %s의 직업은 %s이고 월급은 %i원입니다..\n", name, job, pay);
}
```
해당 강의에서는 소개하지 않았지만 실험해본 결과, 문자열 내에 형식 지정자를 사용한 순서대로 큰 따옴표 뒤에 사용하려는(객체의 타입에 맞게) 변수를 선언해주면 된다.
<br>
<img width="511" alt="image" src="https://github.com/changhwan77/changhwan77.github.io/assets/110464205/f880c323-2a43-47b5-8635-35d49b4625fc">
<br>

### Dart 언어

```dart

void main( ) {
  String name = "창환";
  String job = "Software Engineer";
  int pay = 3000000;
  print("hello $name의 직업은 $job이고 월급은 $pay 입니다.");
}
```
들어가려는 자리에 어떤 변수를 할당할지를 알 수 있기 때문에 다트 언어는 더 직관적인 것 같다. 

## 조건문(branch)과 반복문(loop)

해당 문법은 dart와 c가 큰 차이가 없었다. 예시로, 사용자가 입력한 정수가 짝수인지를 출력하는 간단한 프로그램을 짜보자.

```c
#include <stdio.h>
#include <cs50.h>

int main(void) {
    int n;
    n = get_int("정수를 입력하세요: ");
    if (n%2 == 0) {
      printf("%i는 짝수입니다.\n", n);
    } else {
      printf("%i는 홀수입니다.\n", n);
    }
}
```
<br>
위 코드에서 눈여겨볼 점은 아무 값도 할당하지 않은 n이라는 변수를 미리 선언할 수 있다는 점이다. 그리고 해당 변수가 선언됐으므로 아래 line에서는 변수의 타입을 선언해주지 않고도 에러가 발생하지 않는 것을 볼 수 있다.
<br>
<img width="565" alt="image" src="https://github.com/changhwan77/changhwan77.github.io/assets/110464205/20c100cf-5239-462f-a092-5c35940bea47">
<br>
위처럼 값을 할당하지 않고 변수를 선언하면 어떻게 될지 궁금해서 int n 코드에 breakpoint를 찍고 들여다본 결과다. 정확한 건 아니지만 아마도 integer라는 타입의 n이라는 변수가 메모리 location을 할당 받고 값을 받을 준비를 하고 있는 것 같아 보인다.
<img width="226" alt="image" src="https://github.com/changhwan77/changhwan77.github.io/assets/110464205/9e23d6b2-101a-4b98-a194-567bd08df80c">
그리고 출력값은 위 화면처럼 15를 입력하면 홀수라고 판정되는 간단한 프로그램이었다. 

여기서 get_int는 cs50 파일에 들어있는 함수로 사용자로부터 int값을 입력받아 해당 값을 return하는 역할을 한다.

### Do-While 루프
while, for문은 많이 접하게 되는데 do-while은 생소했다. 

예를 들어, 사용자가 양의 정수를 입력할 시 해당 값에 맞는 nxn 배열을 출력하는 간단한 프로그램을 만든다고 해보자.
<br>
```c
#include <stdio.h>
#include <cs50.h>

int printMatrix(void);

int main(void) {
  printMatrix();
}

void printMatrix (void) {
  int n;
  do
  {
    n = get_int("정수를 입력하세요: ");
  }
  while(n < 1);
  for (int i = 0; i < n; i++) { 
    for (int j = 0; j < n; j++) { 
      printf("#\n");
    } 
    printf("\n");
  } 
}
```
<br>

do-while은 while과 달리, do 뒤에 따라오는 중괄호 { } 안의 instruction이 한 번 실행된 후에 while문으로 가서 조건을 체크하고 while문의 경우에는 조건을 체크하고 loop를 돌린다.
<br>
그리고 특이한 점은 dart의 경우에는 위 코드를 똑같이 실행할 때 printMatrix를 main함수 위에 함수의 시그니처를 선언을 해주지 않아도 에러가 발생하지 않지만 c언어의 경우에는 이를 선언해주지 않으면 아래 정의되어있는 함수까지 읽지 않아서 에러를 발생하게 된다는 점이 특이했다.
<br>
위 printMatrix함수를 설명하자면 n이라는 정수 타입의 변수를 선언해놓고 do 본문에서 사용자로부터 정수 값을 입력받는다.
<br>
이 때, while 에서 조건을 체크하는데 사용자로 부터 받은 n에 할당된 정수 값이 1보다 작다면(true) 다시 do로 올라가서 정수값을 받고 1보다 크다면(false) loop를 빠져나와 아래 for문을 돌며 배열을 출력하게 된다.
<br>
<img width="218" alt="image" src="https://github.com/changhwan77/changhwan77.github.io/assets/110464205/105da435-afab-47f2-be73-77eb8b4c6b1b">

### 하드웨어의 한계

메모리의 location은 유한하기 때문에 우리가 할당할 수 있는 bit수보다 값이 커지게 되면 값이 0으로 초기화되고 오류가 발생하게 된다. 
이와 관련해서는 더욱 잘 알게되면 다시 보충하도록 하겠다!

C언어를 사용하는 문법에 대해 간략히 알아봤다. 

혹여나 틀렸거나 질문이 있다면 피드백 부탁드립니다!
