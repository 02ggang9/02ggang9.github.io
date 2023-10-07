---
published: true
title: "Effective Java Item 3 - 생성자나 열거 타입으로 싱글턴임을 보증하라"
categories:
  - Java
---

생성자를 private로 만들고 public static final 필드로 인스턴스를 만들어높음

오퍼레이션 자체가 외부 API 거쳐서 호출하는 것이 일수도 있고 연산이 오래 걸리는 경우 굉장히 비효율적이다. 만약 인터페이스가 있다면 가짜 대역을 만들 수 있음.
-> 근데 모킹하면 되는거 아닌가? 좀 더 찾아봐야 겠다.

간결함이 장점이다.

---
리플랙션 -> 싱글턴이 깨짐
getDeclaredConstructor -> 선언되어 있는 기본생성자라서 접근 지시지와 상관없이 private 생성자도 접근 가능함.
setAccessible(true)로 설정을 하지 않으면 private여서 생성자를 호출할 수 없게 된다.

getConstructor -> 그냥 public 한 생성자에만 접근할 수 있음

~~~java
private static boolean created;

private Elvis() {
    if (created) {
        throw new UnsupportedOperationException("can't be created by constructor");
    }
}

~~~

static도 다시 한번 공부 ㄱㄱ

그래서 리플렉션 쓰면 싱글턴이 깨지고 private 생성자로 막아야 함. 그런데 이렇게 하면 첫번째 장점이였던 간결함이 사라짐.

두 번째 문제는 직렬화, 역직렬화 하면 여러 인스턴스가 생성된다.

직렬화 역직렬화를 쓰려면 Serializable을 구현해야 함. 

그리고 직렬화 할 때 생성자가 사용이 된다? -> 이건 더 찾아봐야 할 듯.

readResolve() -> 더 찾아봐야 할듯. 오버라이딩 아닌 오버라이딩 같은 메서드??
readResolve()를 재정의 해서 기존에 있던 인스턴스를 반환하도록 만드는 것이다.

---

~~~java
public static final INSTANCE = new Elvis();
private static final INSTANCE = new Elvis();

public static Elvis getInstance() {
    return INSTANCE;
}
~~~

단점은 테스트, 리플렉션, 직렬화역직렬화 똑같은 문제를 가지고 있음. 해결책은 전 단계에서 다 해결했음.

장점은 클라이언트 코드가 getInstace()를 계속 쓰면서 행위, 동작을 바꿀 수 있음.

클라이언트 코드는 바뀌지 않고 새로운 인스턴스를 만들도록 할 수 있음. 클라이언트 코드를 덜 건든다.

~~~java
public static Elvis getInstance() {
    rturn new Elvis();
}
~~~

정적 팩터리를 제네릭 싱글턴 팩토리로 만들 수 있다.
인스턴스는 같지만 타입은 다르게 쓸 수 있음(원하는 타입으로 형 변환을 할 수 있다) equals를 사용하면 두 비교가 같음. 그런데 == 연산자를 사용하면 타입이 다르기 때문에 false가 뜬다.

래퍼런스가 같고 같은 객체임에도 불구하고 (해쉬코드도 같음) 그럼에도 == 비교는 할 수 없다.

Q. 여기에는 왜 T가 한번 더 들어갔을까요? -> 이걸 잘 설명해야 제네릭을 이해한거임.
비록 T 라는 타입 이름은 같지만 스콥이 다름 static 스콥.
~~~java
public static <T> MetaElvis<T> getInstance() {
    return (MetaElvis<T>) INSTANCE;
}
~~~

장점 3. 메서드 참조를 공급자로 사용할 수 있다.

---

열거 타입으로 싱글턴임을 보장하라.

생성자를 private로 만들 필요도 없고 public, static, field를 정의안해도 됨.

리플랙션, 직렬화, 역직렬화에 굉장히 안정적이다.

enum은 태생, 근본적으로 new 이렇게 만들 수 없게 함. 리플렉션 API를 통해서 enum에 컨스럭터를 가져오려 해도 getConstructor 이렇게 가져오려고 해도 내부적으로 막아놓음.

---
인스턴스 메서드 vs 스테릭 메서드 
