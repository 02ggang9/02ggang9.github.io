---
title:  "Effective Java - 열거타입"
categories:
  - Java
---

방학 때 속성으로 자바 언어에 대해서 공부했고 프로젝트에 참여했습니다. 프로젝트를 진행하는데 기초가 정말 부족하다는 느낌을 많이 받았습니다. 그래서 전공 수업으로 플랫폼기반 프로그래밍 수업(JAVA)을 선택했고 Effective Java 강의와 병행하기로 마음먹었습니다.

아래는 백기선 강사님의 "Effiective Java Item 1 - 생성자 대신 정적 팩터리 메서드를 고려해라" 강의 중 부가적인 설명을 정리한 내용입니다.


## 열거타입의 정의와 장점
enums는 Java 언어에서 열거형이라 불리고 특별한 데이터 타입입니다.

enums의 장점은 TypeSafe를 보장하고 가독성이 높아집니다. enum이 없다면 int 값을 쓰거나 메모리를 더 적게 사용하는 char, short를 사용해 현재 상태를 나타낼 수 있습니다. 

~~~java
public class OrderStatus {
    // 0 - 주문 받음
    // 1 - 준비 중
    // 2 - 배송 중
    // 3 - 배송완료
    private int status;

    private final int PREPARING = 1;
    private final int SHIPPING = 2;
}
~~~

이렇게 위처럼 서로 협의된 int 값을 활용해 enum을 대체할 수 있습니다. 하지만 이런 방법은 typeSafe를 보장하지 못합니다. status의 타입은 int 이기 때문에 100이라는 값이 들어올 수 있고 음수 값도 들어올 수 있습니다. 그렇기 때문에 매번 이 값이 valid한지 체크해주는 코드가 들어가야 하고 가독성도 떨어지게 됩니다.

enum을 사용하게 된다면 enum class가 정의한 값들만 사용할 수 있기 때문에 typeSafe를 보장할 수 있습니다. 또, java의 enum은 c와는 다르게 타입까지 체크하기 때문에 더욱 타입에 안전해집니다.

<br>

## Q1. Enum 타입을 가질 수 있는 모든 값을 순회하면서 출력하는 방법은?
이 질문의 의도는 "values() 메서드에 대해서 알고 있는가?" 입니다. values() 메서드는 모든 열거형이 가지고 있는 메서드이기 때문에 컴파일러가 자동으로 값을 추가합니다.

~~~java
public class Order {
    OrderStatus orderStatus = new OrderStatus();
    Arrays.stream(orderStatus.values()).forEach(System.out::println);
}
~~~

<br>

## Q2. 자바 Enum은 생성자와 메서드 필드를 가질 수 있는가?
Enum을 쓰지만 Enum 내부적으로 필드랑 메서드를 가져야 하는 경우도 있습니다. PREPARING(0), SHIPPED(1) 등등 DB에 저장할 때 문자열 그대로 저장하는 것이 아니라 숫자 값으로 변경해 저장할 수 있습니다. 이렇게 하는 이유는 이후 name이 바뀌더라도 DB에 저장되어 있는 숫자 값은 그대로 유지할 수 잇기 때문입니다.

KEEPER R2를 진행할 때 Enum 생성자를 사용하는 방법과 @RequiredArgsConstructor을 사용하는 방법이 있었습니다. 생성자를 사용할 경우 생기는 장점은 lombok에 의존적이지 않고 명시적입니다. 단점으로는 필드가 많으면 복잡해지고 유지 보수성이 떨어집니다. 어노테이션을 사용할 경우 생기는 장점은 유지 보수성이 올라갑니다. 단점으로는 lombok에 의존적이고 명시적이지가 않습니다.

팀이 lombok에 의존적이지 않게 작성한다면 생성자를 사용하는 방법을 사용하고 제한이 없다면 개인 취향에 따라 방법을 선택하시면 됩니다.

~~~java
// 생성자를 사용하는 경우
public enum StatusEnum {
    준비중(1),
    배송중(2),
    배송완료(3),
    ;

    @Getter
    private final long id;

    Status(long id) {
        this.id = id;
    }
}
~~~

~~~java
// 어노테이션을 사용하는 경우
@Getter
@RequiredArgsConstructor
@NoArgsConstructor(access = PROTECTED)
public enum StatusEnum {
    준비중(1),
    배송중(2),
    배송완료(3),
    ;

    private final long id;
}
~~~

<br>

## Q3. equals() vs == 연산자
== 연산자 사용을 권장합니다. 여러가지 이유가 있는데 첫번째로 주소 값을 비교하기 때문에 NPE이 발생하지 않습니다. 두번째로 equals 메서드는 내부적으로 ==을 사용합니다. 따라서 괜히 메서드 호출로 자원을 낭비하지 말고 == 연산자를 사용하는 것이 좋습니다. 

~~~java
// equals는 내부적으로 == 연산자를 사용한다.
/**
    * Returns true if the specified object is equal to this
    * enum constant.
    *
    * @param other the object to be compared for equality with this object.
    * @return  true if the specified object is equal to this
    *          enum constant.
    */
public final boolean equals(Object other) {
    return this==other;
}
~~~

<br>


## Q4. EnumMap과 EnumSet vs HashMap과 HashSet
Enum을 Key로 사용하는 Map이나 Enum만 담고 있는 Set을 선언할 때 EnumMap과 EnumSet을 사용하는 것이 HashMap과 HashSet보다 훨씬 빠릅니다.

EnumMap의 경우 Key 개수로 배열의 길이를 설정하기 때문에 메모리 효율적으로 관리할 수 있고 참조가 매우 빠릅니다. 반면에 HashMap은 해싱 처리과정, Null값 허용, linkedList 참조를 하기 때문에 EnumMap에 비해 느립니다.

~~~java
// enumMap 내부
public EnumMap(Class<K> keyType) {
        this.keyType = keyType;
        keyUniverse = getKeyUniverse(keyType);
        vals = new Object[keyUniverse.length];
    }
~~~

<br>

EnumSet은 value값이 있는지 없는지만 확인하면 되기 때문에 비트 벡터를 사용할 수 있습니다. 반면에 HashSet은 더미 값을 채우기 때문에 EnumSet에 비해 성능이 좋지 않습니다.


<br>

## 마치며
Effective Java 강의를 듣고나서 Enum에 대해서 더욱 잘 알게 되었습니다. 다음은 플라이웨이트 패턴과 인터페이스와 정적 메서드에 대해서 알아보도록 하겠습니다.
