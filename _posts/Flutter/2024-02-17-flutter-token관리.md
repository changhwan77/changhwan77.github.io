---
layout: single
title: Riverpod으로 토큰값 관리하기 w.flutter
toc: true
date: 2024-02-17
categories:
  - Flutter
---

오늘은 유저가 로그아웃 시 홈탭으로 이동하게 한 후 유저의 로그인 상태(토큰 여부)에 따라 로그인 버튼을 띄우는 작업을 진행했어요!

![image](https://github.com/changhwan77/changhwan77.github.io/assets/110464205/5157adfd-4b5f-4b7c-a760-01a5ba621df0)

즉, 로그아웃 버튼을 누르면 토큰이 제거되고 토큰이 제거된 상태를 HomeScreen에 넘겨줘 이를 감지해 로그인 버튼을 화면에서 그리도록 하는 것이 to-be에요.

따라서 다음과 같이 로직을 구성해봤는데요.

![image](https://github.com/changhwan77/changhwan77.github.io/assets/110464205/3f38360b-8db8-4044-b93d-a07fe4fc58dd)

1. 토큰 값이 있는지, 없는지로 로그인 버튼을 그릴지 말지를 판단하기 때문에 상수값의 집합 => Enum 을 사용하기로 했습니다.
2. 단순한 상태 값을 관리한다면 리버팟 공식문서에서는 StateProvider 사용을 권장하기에 이 객체를 사용하기로 결정했습니다.

따라서 유저가 로그아웃을 버튼을 클릭했을 때 토큰 값을 변경시켜주고

```dart
ref.read(tokenProvider.notifier).state = TokenState.absent;
```

홈탭의 Header 위젯에서 토큰 값이 변경됐음을 감지하고 위젯을 rebuild하여 로그인 버튼을 그리도록 로직을 수정한다.

이 때 문제는 앞서 getAuthToken이라는 함수로 접속한 유저의 토큰이 localStorage에 저장되어 있는지 확인하고 (비동기)

1. 데이터가 존재하고

2. 데이터가 null이거나 비어있으면

로그인 버튼을 return 하도록 로직이 설정되어 있었다.

따라서 로그인 버튼이 표시되는 case는 유저의 토큰이 없는 경우이기 때문에 아래와 같이

1. tokenState에 tokenProvider의 값을 watch하고

2. 변경된 tokenState의 값이 TokenState.absent인 경우를 if 문에 추가

```dart
FutureBuilder<String?>(
            future: _getAuthToken(),
            builder: (context, snapshot) {
              if (snapshot.connectionState == ConnectionState.done) {
                if (snapshot.hasError) {
                  //TODO: 에러 발생 시 어떻게 처리할 것인지 추가 필요
                }
                if (snapshot.hasData) {
                  // 토큰 유무 상태가 변경될 시 header 부분 rebuild
                  TokenState tokenState = ref.watch(tokenProvider);

                  if (snapshot.data!.isEmpty ||
                      snapshot.data == null ||
                      tokenState == TokenState.absent) {
                    return TuTiButton(
                      padding: EdgeInsets.symmetric(
                          vertical: 10.h, horizontal: 25.w),
                      fontSize: 10.sp,
                      title: '로그인',
                      onPressed: () => _showLoginDialog(context),
                    );
                  } else {
                    // 토큰이 있을 시 tokenProvider의 값을 토큰 있음으로 변경
                    ref.read(tokenProvider.notifier).state = TokenState.present;
                  }
                }
              } else {
                return const RSizedBox();
              }
              return const RSizedBox();
            },
          ),
```

그리고 추가적으로 유저의 토큰이 null이 아니고 비어있지 않다면 토큰 프로바이더의 상태를 present로 변경해줌으로써 구현을 완료했습니다!

### 업데이트

고객 피드백을 위해 PR을 통해 Preview 배포를 통해 해당 버전을 봤는데, 로그아웃 후 로그인 상태가 반영이 즉각 안되는 에러가 있었음.
따라서 token값을 직접 관리하는 StateProvider를 이용하기로 했고 로직은 다음과 같았음.

![image](https://github.com/changhwan77/changhwan77.github.io/assets/110464205/3b65dbb0-1a9c-46a9-ae5c-7b0790b5caf0)

1. 로그인 시 local Storage에 저장된 token을 불러와 tokenProvider에 저장.
2. 로그아웃 시
    1. 로컬 스토리지의 토큰을 제거
    2. tokenProvider의 상태를 빈 문자열로 변경
3. 메인 탭에서 tokenProvider를 watch해 상태가 null이거나 빈 문자열이면 로그인 버튼이 뜨도록 로직 설정.

위와 같이 구현하여 에러를 해결했습니다!
