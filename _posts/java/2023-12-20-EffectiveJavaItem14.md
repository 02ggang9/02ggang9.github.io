---
published: false
title: "EffectiveJava Item 14 - Comparable을 구현할지 고민하라"
categories:
  - Java
---

## 규약
컴페어러블
순서가 있는 경우 순서를 정해주는 방법을 정의해 줄 수 있음
컴파일 시점에 타입 체킹이 가능함
Equals에 더해서 제네릭을 지원한다.

BigDecimal -> 컴페어러블을 구현하고 있음
반사성

빅데시멀의 compareTo는 -1,0,1을 반환하지만 인터페이스 스펙은 -1,0 뿐이다. 따라서 1이 나오는 걸 예상하고 코드를 작성하면 안된다.

추이성

일관성 -> 동일한 인스턴스를 여러번 비교해도 같은 결과가 나와야 한다.
값이 같으면 

compareTo -> 스케일 정보를 버리는 거 같음 -> 이건 좀 더 찾아봐야 할 듯.
equals -> 스케일 정보도 중요하게 생각함 -> 2.0과 2.00은 다르다고 생각함. 따라서 이건 문서화 하라고 책에서 추천함.

## 구현 방법
제네릭임.

오버라이딩 할 때 오버라이딩을 붙여줘야함. 안 붙여주면 컴파일 시점에 확인이 제대로 안된다.

compare

여러 필드가 있는 경우..
(추천하지 않음)
상속을 쓴다고 하면 comparable을 재 정의할 수 없음...
다형성 적용이 안됨 -> 오버라이딩 된 것만 다형성 적용이 됨..
별도의 Ci=omparator를 재정의해서 할 수 있음..

(추천) 컴포짓을 사용해라


## 

Comparator를 만들어라

