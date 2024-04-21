---
layout: single
title: (CS50) - 배열
toc: true
date: 2024-04-21
categories:
  - CS50
tag: [C언어]
---

## 컴파일링(Compiling)

clang 을 이용해 우리가 작성한 소스 코드를 컴파일하여 오브젝트 코드를 생성하여 이 파일을 실행해 우리가 원하는 출력을 볼 수 있었다. 

그렇다면 컴파일링은 source code가 object(machine) code로 변환된다는 것인데 더 구체적으로는 어떤 과정을 거치게 될까? 라는 물음이 나올 수 있다. 

컴파일 과정을 총 4단계를 거치는데 크게는 다음과 같다.

1. 전처리(Preprocessing), 2. 컴파일링 3. 어셈블링 4. linking

### Preprocessing
첫 번째는 전처리 단계로 예를 들어, 우리가 컴파일러에게 #include 에 포함된 파일을 봐달라고 얘기하는데 컴파일러는 소스 코드를 읽으면서 #include에 포함된 파일 중 정의되어 있는 코드들을 참고하여 우리가 작성한 소스 코드에 들고 와서 replace한다.

### Compiling
바로 기계어인 binary로 변환되는 게 아니라 assembly code로 변환하게 된다. 해당 코드는 기계어가 의미하는 것을 symbolic하게 표현하여 추상화시킨 언어인데 소스 코드를 전부 어셈블리 코드로 변환함으로써 기계어로 변환할 준비를 마치게 된다.

### Assembling
정확한 건 아니지만, 아마 여기서 어셈블러가 개입되지 않을까 싶다. 어셈블러가 컴파일러에 의해 변환된 어셈블리 코드를 해석해 매칭되는 기계어로 변환한다.

### Linking
이 개념은 생소했는데, 예를 들어 printf 함수를 사용하려면 우리는 클라우드 환경에 있는 stdio.h 라는 파일에 정의되어 있는 printf 함수를 가져와야 하고 이를 컴파일러에게 알려줬었다. 이렇게 우리가 사용하려는 외부 라이브러리의 코드를 기계어로 변환하고 우리가 작성한 소스 코드와 **연결(link)**해야 우리가 원하는 대로 프로그램이 실행된다는 것이다.

이렇게 기계어로 변환, linking 과정까지 거치게 되면 최종 object 파일이 생성되고 컴퓨터는 이 파일을 실행하게 되는 것이다.

## 디버깅(Debugging)
디버깅은 우리가 프로그램을 만드는데 예기치 못한 에러가 발생할 수도 있다. 이 때 발생한 에러를 차근차근 되짚어가며 어떤 부분에서 에러가 발생했는지 원인을 파악하는 작업이다. 

cs50에서는 자체 라이브러리에서 제공하는 help50, debug50, style50이라는 명령어를 제공하는데 debug50을 이용해 디버깅을 진행해봤다. 

### Printf 함수 사용
원래 하던 방식인 print를 찍어서 내가 생각하는 지점의 state가 무엇인지 콘솔에 출력해보는 것이다. 하지만 이 방식은 코드가 지저분해지거나 하드 코딩이 필요한 작업으로 더 나은 방법이 있었다.

### BreakPoint 사용

> BreakPoint란? 
- 내가 특정 시점에서 프로그램의 state가 어떤지 보기를 원하는 지점(point)
> 

아래는 원하는 프로그램의 state를 보기를 원하는 지점에 breakpoint를 찍고 cs50에서 제공하는 debug50 (실행할 파일 path)를 프롬프트에 입력했을 때 나오는 이미지다.
<br><br>
![image](https://github.com/changhwan77/changhwan77.github.io/assets/110464205/0f903f0e-7d78-4dc8-a8b9-e060e4baa91b)
<br><br>
위 예시는 기호 #(해시)를 10번 출력하기 위해 프로그램을 작성했는데 11개의 해시가 출력되어 어디서 잘못됐는지 알기 위해 Line 6 코드에 breakpoint를 찍고 debug시 내가 코드를 한 줄씩 실행해보면서 변수 i에 할당된 값이 어떻게 변경되는지 알 수 있었다. 

## 코드의 디자인(Design of Code)
소프트웨어를 작성할 때 “내가 시간이 흐른 후에 돌아왔을 때도 작성한 코드가 무엇인지 알 수 있어야 하고 같은 동료가 봐도 읽기 쉬운 코드를 작성하고 유지보수가 쉬운 코드”를 작성해야 한다.
<br>
이를 위해서는 cs50에서 2가지 방법을 소개 했다.<br><br>
1. 테스트 코드 작성
    - 예를 들어 기존에 돌아가던 제품이 있는 팀에 합류하는 경우 내가 작성하는 코드에서 오류가 발생하면 안되기 때문에 내 코드가 정확한지 확인하는 테스트 코드를 작성해야 한다.
2. 고무 오리
    - 스스로에게 내가 코드를 어떤 식으로 짰는지 단계 단계 설명하며 문제점을 발견하는 방식이다.
<br><br>
결국은 우리는 1인 개발자가 될 것이 아니라면 또는 1인 개발자라도 소스 코드가 오픈될 수 있는데 타인이 내 코드를 봤을 때 읽기 쉽고 추후에 변경 사항이 일어날 텐데 이러한 변경 사항이 다른 범위에 영향을 최대한 작게 코드를 작성하는 것이 바람직하겠다는 생각이 들었다. 

## 배열
<br>
만약에 중간 고사 점수의 평균을 내는 프로그램을 작성한다고 해보자. 

```c
#include <stdio.h>
#include <cs50.h>

int main(void)
{
  int score1 = 72;
  int score2 = 43;
  int score3 = 67;
  
  printf("average: %i\n", (score1 + score2 + score3) / 3); // 평균 점수 출력
}
```

위에 작성한 코드의 문제점은 score1부터 3까지 전부 int 타입으로 같은 형태의 변수와 값을 반복하고 있다는 것이다. **즉, 우리가 입력해야 하는 점수가 한 학년(300명)이라고 하면 우리는 300줄의 코드를 작성해야 할 것이다**. 우리는 이를 해결하기 위해 배열(Array)라는 개념을 사용하여 다음과 같이 코드를 작성할 수 있다. 

```c
#include <stdio.h>
#include <cs50.h>

int main(void)
{
  int scores[3]; // 배열의 길이 할당
  scores[0] = 72;
  scores[1] = 54;
  scores[2] = 67;  
  int sum = scores[0] + scores[1] + scores[2]
  printf("average: %i\n", (sum / 3)); // 평균 점수 출력
}
```

즉, 우리가 값을 저장하는 memory location을 상상해보자. 그럼 score1, score2, score3으로 다른 변수로 나눠서 저장되어있던 점수들이 score[0], score[1], score[2]로 하나의 배열로 메모리에  이어서 저장되어 하나의 변수에 할당하여 관리하게 된 것이다. 

### 배열을 이용해 사용자 입력에 따른 평균 시험 점수 출력 프로그램 만들기

```c
#include <stdio.h>
#include <cs50.h>

int main(void)
{
	// 배열의 길이 할당(입력할 과목 수 받음)
	int n = get_int("Number of Scores: ");
  int scores[n];
  
  // 배열에 요소 할당(과목별 점수 할당)
  for (int i = 0; i < n; i++)
  {
	  scores[i] = get_int("%i번째 과목의 시험 점수를 입력해주세요: ", i + 1);
  }
  // 평균 점수 출력
  printf("average: %.1f\n", average(n, scores));
}

float average(int length, int array[])
{
	int sum = 0;
	for (int i = 0; i < length; i++)
	{
		sum = sum + array[i];
	}
	return sum / (float) length; 
}
```

출력 결과 

<img width="644" alt="image" src="https://github.com/changhwan77/changhwan77.github.io/assets/110464205/13cbae21-f85b-4889-b6d5-aa2432032119">

### 예상과 달랐던 점
<br>
여기서 내 예상과 달랐던 점은 배열을 생성할 때였다. 즉, int scores[3] 라고 작성하면 index가 0, 1, 2, 3 해서 총 4개의 요소를 가진 배열이 생성되는 줄 알고 작성했지만 컴파일링하려고 시도했는데 오류가 발생했다. 
<br><br>
> C언어에서 배열을 처음 생성할 때 int scores[N]에 들어가는 N은 배열의 길이를 의미하는 것이다.
즉, int scores[3]이라고 하면 3개의 요소를 가지는 배열을 생성한다는 의미다.
> 
<br><br>
그리고 위 평균 값 출력 프로그램을 실행하면 에러가 하나 발생할 것입니다. 왜냐하면 제가 컴파일하려고 했더니 에러가 발생했거든요. 이유는 다음 부분입니다! 
<br>
> **함수의 프로토타입 선언해주기**
float average(int length, int array[]);
> 

왜냐하면 c언어에서는 main함수 밑에 우리가 정의한 함수를 선언까지 읽지 않기 때문입니다. 따라서 main함수 위에 함수의 프로토타입이라고 설명하던데(아마도 함수의 시그니처(name, argument)) 를 선언해주면 컴파일러가 이런 함수가 아래에 있음을 알고 우리가 정의한 함수까지 스캔하게 되어 컴파일 에러가 발생하지 않게 되는 것입니다.

## 문자열
문자’열’은 문자(Character)로 이루어진 배열이다.<br>
문자(Character) 하나 당 1바이트의 메모리 공간을 차지하게 되는데요. 위에서 배운 배열처럼 그런 문자들의 배열을 말하며 배열의 속성과 같이 문자열은 index로 각 문자에 접근할 수 있습니다. 

### Null Terminated String(’\0’)
위까진 유추할 수 있지만 유념해야 할 점은 만약에 **컴퓨터가 메모리에 저장된 문자열을 읽으면서 해당 문자열의 끝을 알 수 없기 때문에 이를 위해 만든 장치**가 널 종결 문자(’\0’) 입니다.<br>
즉, 우리가 jude 라는 문자열을 메모리에 저장한다면 우리가 예상하는 길이 4개의 문자가 아니라 +1인 5개의 문자로 이루어진 문자열이 됩니다. 
<br><br>
```c
#include <stdio.h>
#include <cs50.h>

int main(void) {
    string names[4];
    names[0] = "hi";
    names[1] = "aa";
    names[2] = "bb";
    names[3] = "jude";
		
		//문자열 jude의 5번째 index(널 종결 문자)에 접근
    printf("%i\n", names[3][4]);
}
```

출력 결과
<br><br>
<img width="264" alt="image" src="https://github.com/changhwan77/changhwan77.github.io/assets/110464205/090d16bc-b35e-4a5d-beec-882c386a0336">

이렇게 배열에 대해서 알아봤는데요!<br>
끝까지 읽어주셔서 감사드리며 궁금한 점 또는 피드백은 편하게 댓글 부탁드립니다 :-)
