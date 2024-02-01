---
layout: single
title: 2024-01-31 TIL (플러터 텍스트 인풋 및 키보드 사용성 향상)
toc: true
date: 2024-01-31
---

### 😘 오늘의 나는 무엇을 잘했나요?

---

- 오늘도 계속 배우는 부분을 메타인지 보드에 작성하며 머릿속 노드들이 연결되도록 노력했다.

### 😆 오늘의 일에서 나는 어떤 것을 배웠나요?

---

- TextField에 유저가 값을 입력할 때 값을 가져오고 값이 변경 시 추적하는 법
    - 값을 가져오는 법
        - textEditingController를 사용한다.
            1. initState에서 컨트롤러를 초기화하고 
            2. dispose에 컨트롤러를 dispose한다. (컨트롤러가 쓰이지 않을 때는 값을 홀딩하고 있는 것을 버리기 위함.)
            3. textField 클래스의 controller 프로퍼티에 할당한다.
            4. 유저가 입력한 값을 controller.text 로 값을 받는다!
    - 값 변경 시 추적하는 법
        - TextField의 onChanged와 controller로 가능.
            - onChange는 파라미터를 text로 받고 이를 추적하면 되고
            - controller는 컨트롤러 객체가 addListener 메서드를 가지고 있는데, 이를 initState에서 초기화 하면 된다.
                
                addListener의 파라미터에는 해당 값을 어떻게 추적할 건지에 대한 표현식을 작성하면 된다.
                
    - 컨트롤러 주의 사항으로는 꼭 dispose를 해야한다는 것이다! 컨트롤러가 사용되지 않을 때는 이를 버려야 컨트롤러가 사용하는 리소스가 세는 것을 방지할 수 있다.
- TextField 스타일링 방법
    - decoration 프로퍼티에 InputDecoration 객체를 사용한다.
        - InputDecoration에는 border, focusBorder, pre(suf)fix icon, labelText, hintText 등을 설정할 수 있다.
- keyBoard가 올라오는 것을 다루는 법
    - 유저가 텍스트필드가 있는 화면에 진입 시
        - 자동으로 키보드가 뜨게 할거냐 → autoFocus: true로 설정하면 된다.
        - 수동으로 할거냐
            - FocusNode 객체를 사용한다.
                - initState, dispose 둘 다에서 선언해줘야 하고
                - 포커스가 필요한 부분에 FocusNode.requestFocus( )를 요청해 Flutter에서 관리하는 Focus Tree에서 해당 포커스로 이동하게 만들어 키보드가 올라오도록 할 수 있다.
- keyBoard가 올라왔을 때 같은 객체 내에서 키보드 이외의 화면을 눌렀을 때 키보드가 사라지게 하는 법
    - 컨텐츠를 담고 있는 전체 객체에 GestureDetector로 감싸준 다음 onTap에 새로운 포커스를 생성해 이전 포커스를 해제하면 된다.
        - `FocusScope.of(context).requestFocus(FocusNode())`;
            - 이는 근처 Focus Tree에서 focus들을 찾고 활성화된 포커스노드에 해제를 요청하는 로직이다.
- 반응형 레이아웃 구현 방법
    - LayoutBuilder → 부모 위젯의 제약 조건을 받아와서 이를 바탕으로 자식 위젯에 제약을 적용할 수 있다.
        - 같이 자주 쓰이는 게 ConstraintedBox로 최소, 최대 / 너비, 높이를 자식 위젯에 적용할 수 있다.
    - MediaQuery.of(context) : 사용자의 디바이스 관련 정보를 받을 수 있다.
        
        ![1](https://github.com/changhwan77/changhwan77.github.io/assets/110464205/b99bdbde-ae1d-42ab-b7a0-b545ff7964a2)
        <br>
        - viewInset : 사용자의 기기(주로 키보드)가 ui를 덮은 공간에 대한 픽셀값
        - padding : safeArea 존인 노치 부분
        - viewPadding : 위와 같은데 차이는 키보드가 올라오면 padding은 0으로 바뀌지만 viewPadding은 키보드 뒤에 가려진 노치 공간에 대한 픽셀값을 계속 인식한다.
- SingleChildScrollView은 자식 컨텐츠가 제약 조건을 넘어서야만 스크롤이 작동한다.

### 🤢 오늘의 나는 어떤 문제를 겪었나요?

---

- 키보드가 올라왔을 때 높이 조절하는 법
    
    ![2](https://github.com/changhwan77/changhwan77.github.io/assets/110464205/0aae6341-b13d-4d2e-a436-cd33a0f37223)
    <br>
    위 사진과 같이 텍스트필드를 클릭 시 키보드가 올라와서 화면을 다 가리게 되는 문제가 있었다.
    
    여기에 대한 솔루션이 스크롤을 입혀 키보드 위로 컨텐츠가 스크롤이 가능하도록 하여 키보드 뒤에 숨은 컨텐츠도 볼 수 있도록 하는 것이었는데, 
    
    SingleChildScrollView 위젯을 사용했는데, 이 위젯은 자식 위젯에게 최대한 많은 공간을 허용하고 같이 자주 쓰이는 게 Column 위젯인데 Column 위젯은 가능한 많이 높이를 차지하려고 했고
    
    Flexible 위젯도 같이 쓰다보니 높이 제한이 걸리지 않아서 layout이 계속 터지는 문제가 발생했다.
    

### 🤩 앞으로 어떻게 해결할 것인가요?

---

- 레이아웃을 짤 때 최대한 제약을 잘 넘길 수 있는 방법을 계속 고려해야겠다.

### 🙏🏻 오늘의 감사한 점 3 ~ 5가지 작성

---

- 오늘은 일찍 나올 수 있었고 목이 덜 아픈 상태로 오랜기간 지속할 수 있었다.
- 여자친구가 알바가 끝난 기념으로 재밌는 영화도 보고 친구랑 술도 먹고 노래방도 갔다가 행복하게 보내고 온 것 같아 감사하다.
- 오늘도 엄마, 아빠가 행복하게 집에서 있는 것 같아 감사하다.
