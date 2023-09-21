---
title: "Effective Java Item 61 - 박싱된 기본 타입보다는 기본 타입을 사용해라"
categories:
  - Java
---

KEEPER R2 프로젝트 중 MeritType Entity에 is_merit 컬럼이 추가되었고 그 컬럼을 Response DTO에 넣어주는 과정에서 필드의 타입이 기본 타입과 박싱된 기본 타입이 섞여 있는 것을 확인했습니다. 자바의 기본이 부족했던 초반에는 두 타입에 대한 개념이 정확히 없어서 어떤 날에는 전부 기본 타입으로 다른 날에는 전부 박싱된 기본 타입으로 DTO 필드의 타입을 결정했습니다. 

자바프로래밍 수업에서 Boxing과 UnBoxing에 대해서 배웠고 고성능의 애플리케이션을 개발하기 위해서는 이 두 타입의 차이점을 알고 있어야 한다고 말씀하셨습니다.

## 기본 타입 vs 래핑된 기본 타입
기본 타입은 null 값이 들어갈 수 없고 데이터의 값(data)이 들어갑니다. 반면에 래핑된 기본 타입은 null 값이 들어갈 수 있고 데이터의 값이 아니라 주소 값(memory address)이 들어갑니다.


### 식별성
> 박싱된 기본 타입의 두 인스턴스의 값이 같아도 서로 다르다고 식별될 수 있다. (이펙티브 자바 p.358)

위의 말은 아래의 코드로 이해할 수 있습니다.

~~~java
  public static void main(String[] args) {
    System.out.println(new Integer(10) == new Integer(10)); // false
  }
~~~

두 인스턴스의 값은 10으로 동일한 값이지만 래핑된 기본 타입이기 때문에 주소 값을 비교합니다. 따라서 두 인스턴스의 주소 값은 다르기 때문에 값은 false가 되고 서로 다르다고 식별할 수 있습니다. 위의 코드를 이해하면 이펙티브 자바에서 나온 예시 코드를 쉽게 이해할 수 있습니다.

~~~java
public static void main(String[] args) {
    Comparator<Integer> naturalOrder = (i, j) -> (i < j) ? -1 : (i == j ? 0 : 1);
    int result = naturalOrder.compare(new Integer(42), new Integer(42));
    System.out.println(result); // 1
  }
~~~

(i < j)을 비교하는 과정에서는 기본 타입으로 언박싱 후 값을 비교해서 통과하지만 (i == j)을 비교하는 과정에서는 두 인스턴스의 주소 값을 비교하기 때문에 결과 값이 1이 출력되게 됩니다.


### null
> 기본 타입의 값은 언제나 유효하나, 래핑된 기본 타입은 유효하지 않은 값, 즉 null 값을 가질 수 있다. (이펙티브 자바 p.358)

~~~java
public class Wrapping {
  static Integer i;

  public static void main(String[] args) {

    if (i != 42) { // NullPointException
      System.out.println("안녕하세용");
    }
  }
}
~~~
i의 초기값은 null이고 i == 42을 비교할 때 박싱이 자동으로 풀리게 됩니다. null 참조를 언박싱하기 때문에 NullPointException이 발생하게 됩니다. (기본 타입과 래핑된 기본 타입을 혼용한 연산에서는 래핑된 기본 타입이 언박싱됩니다)


### 성능 차이
~~~java
  public static void main(String[] args) {

    long startTime = System.currentTimeMillis();
    long sum = 0L;
    for (Integer i = 0; i < Integer.MAX_VALUE; i++) {
      sum += i;
    }
    long endTime = System.currentTimeMillis();
    System.out.println((endTime - startTime) / 1000.0);
  }
~~~

기본 타입을 사용했을 경우 0.75초가 걸리고 래핑된 기본 타입을 사용했을 경우 4.025초가 걸립니다. 결과에서 알 수 있듯이 언박싱 하는 과정이 포함되면 성능이 느려지는 것을 확인할 수 있습니다.


## 엔티티와 DTO에서는 어떤 타입을 선택해야 할까?
데이터베이스를 설계할 때 not null 제약 조건이라면 기본타입을 사용하고 nullable이면 래핑된 기본 타입을 사용해야 합니다.

JPA에서는 기본 타입에 @Column을 생략하게 된다면 not null 옵션이 기본이고 래핑된 기본 타입이라면 nullable 옵션이 기본입니다. 기본 타입에 @Column을 사용한다면 default 값이 nullable이기 때문에 기본 타입과 @Column을 사용한다면 nullable = false로 지정하는 것이 안전합니다.

DTO에서는 엔티티에서 정의된 타입을 따라해야 합니다. 그렇지 않을 경우 데이터베이스에서 데이터를 가져오고 stream을 통해서 DTO로 변환하는 과정에서 오토 박싱&언박싱 문제로 성능이 저하될 수 있습니다.


## 최적화는 신중히
마지막은 모 선배님 개발자님의 말씀을 이용하면서 마치겠습니다. (항상 감사합니다. 많이 배우고 있습니다~!)

> 그런데, 정말 병목현상이 박싱, 언박싱에서 일어날까? 는 다른 문제입니다. 위 블로그에서도 볼 수 있듯이 for문에서 Integer.MAX_VALUE (약 21억)번을 돌렸을 때 비로소 1~2초 성능차이를 볼 수 있습니다. 과연 애플리케이션 개발에서 그 정도의 객체를 가지고 for문을 돌거나 작업을 할 일이 있을까? 를 생각해보면 아니라는 결론이 나옵니다. </br></br>
지적하신게 성능 문제를 일으키는건 자명하지만, 거기에 너무 매몰되서 생산성이 낮아진다면 고민해 볼 필요는 있을 것 같습니다. (물론 체득되서 자연스럽게 primitive, wrapper 타입을 적재적소에 사용하면 베스트겠지만용!)








