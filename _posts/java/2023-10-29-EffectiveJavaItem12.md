---
published: true
title: "Effective Java Item 12 - toString을 항상 재정의하라"
categories:
  - Java
---


## 모든 구체 클래스에서 toString을 재정의하자.
toString을 재정의해 클래스에 적합한 문자열을 반환해야 합니다. Object의 toString은 아래와 같이 클래스 이름과 '@', 해쉬 코드를 포함하고 있는데 사람이 알아보기 힘든 형태입니다.

~~~java
public String toString() {
    return getClass().getName() + "@" + Integer.toHexString(hashCode());
}

// Object toString 결과
// PhoneNumber@adbbd
~~~

이 때문에 클래스가 가지고 있는 유익한 정보를 적절한 포맷팅 과정을 거쳐 반환하도록 재정의 합시다.

## toString에 모든 정보를 담지 마라
Effective Java 책에서는 반대로 "toString은 그 객체가 가진 주요 정보 모두를 반환하는게 좋다"고 말하고 있습니다. 하지만 백기선님 강의에서 MS, 아마존 등의 회사를 다니셨을 때 로깅조차 남겨서는 안되는 정보가 많이 있다고 하셨습니다(로그 탈취가 됐을 때를 대비) 따라서 책처럼 극단적으로 모든 정보를 반환하지 말고 동료 개발자들과 상호 합의를 거치고 포맷팅을 결정하도록 합시다.

