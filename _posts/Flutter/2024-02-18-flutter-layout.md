---
layout: single
title: ListView.builder를 사용해 UI 구현하기 with.layout
toc: true
date: 2024-02-18
categories:
  - Flutter
---

![image](https://github.com/changhwan77/changhwan77.github.io/assets/110464205/fac5f2da-bb0b-495b-b77b-738b5d90cb95)

이번 포스팅은 ListView.builder를 사용해 위와 같은 화면을 구현하는 방법에 대해 다룰 예정인데,

즉, 저 남색 카드 같이 생긴 것을 상품 카드 객체로 바라보고 카드가 들고 있는 정보들은 서버에서 정보를 받아서 뿌리는 로직을 구현 예정이에요.

아직 서버 api가 구현되지 않은 상태라 임시로 더미 데이터를 만들어서 사용했습니다!

### 카드 객체 모델링하기

위 카드를 봤을 때 반복되는 패턴이 보이시나요? 저는 다음 4가지로 카드 객체가 가질 property를 생각했어요.

1) 상품의 제목, 2) 정가(빨간색 줄) 3) 할인율 4) 할인가

그리고 카드 객체가 하는 행위(method)는 클릭 시 유저가 상품 상세 페이지로 이동하게 만드는 것으로 파악했어요.

**꼭 필요한 프로퍼티 구분하기**
1), 4)는 카드가 생성됨에 있어 꼭 필요하고 2)정가, 3)할인율의 경우에는 있을 수도 있고 없을 수도 있기 때문에 다음과 같이 클래스를 생성해줬어요.

class GoodsCard {
  final String title;
  final double? regularPrice;
  final double? discountRate;
  final double discountedPrice;

  const GoodsCard(
      {required this.title,
      required this.discountedPrice,
      this.discountRate,
      this.regularPrice});
}

## Container 위젯을 활용해 상품 카드 만들기
ListView.builder 위젯을 사용하기 전에 먼저, Container 위젯을 활용해 상품 카드를 만들어줬어요.

### 상품 카드 구현을 미리 생각해보기

![image](https://github.com/changhwan77/changhwan77.github.io/assets/110464205/465f1874-58cc-4762-a1db-290424625cf6)

저는 객체를 만들기 전에 항상 구조를 시각화한 다음 작업에 들어가는데요!

상품 카드는 위와 같이 머릿속에 그려졌어요.

1) 상품 상세 페이지로 이동해야 하는 메서드 구현
이를 해결하기 위해 GestureDetector를 최상위 위젯으로 그리고 onTap에 이동할 화면의 routePath를 받습니다.

2) child에는 Container가 들어가고 컨테이너 모양을 잡기 위해 decoration 프로퍼티와 컨테이너 안에 들어갈 내용을 삽입하기 위해 child 프로퍼티를 사용했어요.

3) 상품 카드의 프로퍼티들은 세로로 나열되어있기 때문에 Column위젯을 사용했고 제목, 정가, 할인율, 가격 순으로 Text위젯을 사용할 예정이에요.

### 실제 Container 구조 잡기

![image](https://github.com/changhwan77/changhwan77.github.io/assets/110464205/e218500c-f05f-457d-af8d-4f6b6c450d83)

1) mainAxisAlignment:

Column 위젯은 세로로 자식들을 나열하기 때문에 mainAxis가 세로인데요! 텍스트들의 위치의 시작이 컨테이너의 제일 위로 안붙고 중앙으로 되어있기 때문에 mainAxisAlignment.center로 설정해요
2) crossAxisAlignment:

Column의 cross는 세로의 반대인 가로이기 때문에 텍스트들을 보면 가로의 시작점에 위치하기 때문에 start로 설정했어요.
3) padding과 margin:

디자이너님이 작업해주신 피그마에 따라 설정해줍니다.
4) 추후 서버에서 각 텍스트에 받을 내용을 해당 위젯에 프로퍼티로 선언해줍니다.

5) 가격의 경우에는 숫자를 원화로 환산해줘야 하기 때문에 intl 패키지를 사용했어요.

6) 정가와 할인율은 null이 될 수도 있기 때문에 이를 처리하기 위해 아래와 같이 if문으로 null일 때와 아닐 때를 설정해줬어요.

### 완성된 상품 카드 코드

```dart
class GoodsContainer extends StatelessWidget {
  const GoodsContainer(
      {super.key,
      required this.title,
      required this.discountedPrice,
      this.discountRate,
      this.regularPrice});
  
  // 상품 카드에 꼭 필요한 것은 null이 될 수 없고 required를 네임드 파라미터에 선언해 해당 프로퍼티에 값을 할당하지 않으면 안되도록 설정.
  final String title;
  final double? regularPrice;
  final double? discountRate;
  final double discountedPrice;

  @override
  Widget build(BuildContext context) {
  // intl 패키지 사용한 부분
  // locale은 국가, symbol은 원화 앞에 보통 KRW를 붙이는데 이를 생략하기 위해 empty로 설정
    final numberFormat = NumberFormat.currency(
      locale: 'ko_KR',
      symbol: '',
    );
    return GestureDetector(
      onTap: () {},
      child: Container(
        width: MediaQuery.of(context).size.width * 0.8,
        margin: EdgeInsets.symmetric(horizontal: 45.w, vertical: 8.h),
        padding: EdgeInsets.symmetric(horizontal: 25.w, vertical: 30.h),
        decoration: BoxDecoration(
          color: ColorConstants.primaryColor,
          borderRadius: BorderRadius.circular(25.sp),
        ),
        child: Column(
          mainAxisAlignment: MainAxisAlignment.center,
          crossAxisAlignment: CrossAxisAlignment.start,
          children: [
            Text(
              title,
              style: Theme.of(context)
                  .textTheme
                  .bodyLarge!
                  .copyWith(color: Colors.white),
            ),
            if (regularPrice != null)
              Text(
                numberFormat.format(regularPrice),
                style: Theme.of(context).textTheme.bodyLarge!.copyWith(
                    color: Colors.white,
                    fontSize: 22.sp,
                    decoration: TextDecoration.lineThrough,
                    decorationColor:
                        ColorConstants.personalBrandingDividerColor,
                    decorationThickness: 2.0),
              ),
            if (regularPrice == null)
              RSizedBox(
                height: 35.w,
              ),
            if (discountRate != null)
              Text(
                '${numberFormat.format(discountRate)}%할인',
                style: Theme.of(context).textTheme.bodyLarge!.copyWith(
                    fontSize: 18.sp,
                    color: ColorConstants.personalBrandingTextHighlightColor),
              ),
            if (discountRate == null)
              RSizedBox(
                height: 35.w,
              ),
            Text(
              numberFormat.format(discountedPrice),
              style: Theme.of(context).textTheme.bodyLarge!.copyWith(
                  color: Colors.white,
                  fontSize: 35.sp,
                  fontWeight: FontWeight.w900),
            ),
          ],
        ),
      ),
    );
```

## ListView.builder로 상품 카드 동적으로 생성하기

ListView.builder 위젯은 itemCount 프로퍼티와 itemBuilder 메서드에 값을 꼭 할당해줘야 해요.
왜냐하면 1) 몇개의 자식을 생성할 것인지(itemCount) 2) 그 자식들을 어떻게 화면에 그릴 것인지(itemBuilder)를 선언해줘야 하거든요.

>
여기서 잠깐! ListView.builder의 작동 방식 알아보기

- itemBuilder가 자식을 그리는 방식이 itemCount에 선언한 개수만큼 0부터 index(itemCount개수)까지 순회하며 itemCount 개수만큼의 위젯을 생성해요.
>

따라서 상품 카드를 만들 것이기 때문에 다음과 같이 더미 데이터를 생성해줍니다.

```dart
List<GoodsCard> goodsCards = const [
  GoodsCard(
      title: 'aaaaaaaa',
      regularPrice: 600000,
      discountRate: 50,
      discountedPrice: 300000),
  GoodsCard(title: 'xxxx', discountedPrice: 250000),
  GoodsCard(title: 'xxxxx', discountedPrice: 250000),
  GoodsCard(title: 'yyyyyyy', discountedPrice: 250000),
];
```

즉, index를 순회하며 자식 위젯을 만들기 때문에 List라는 데이터 형식을 사용했어요.

예시:
```dart
goodsCards[1] = GoodsCard(title: 'xxxx', discountedPrice: 250000)
```

### ListView.builder 사용한 코드

```dart
ListView.builder(
              itemCount: goodsCards.length,
              itemBuilder: (BuildContext context, index) {
                return GoodsContainer(
                    title: goodsCards[index].title,
                    regularPrice: goodsCards[index].regularPrice,
                    discountRate: goodsCards[index].discountRate,
                    discountedPrice: goodsCards[index].discountedPrice);
              },
            ),
```

이렇게 서버 api가 만들어진 후에 바로 연동할 수 있게 상품 카드를 만들어봤습니다!

긴 글 읽어주셔서 감사합니다 :-)
