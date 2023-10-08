---
published: true
title: "Effective Java Item 4 - 인스턴스화를 막으려거든 private 생성자를 사용하라"
categories:
  - Java
---

## Abstract를 쓰면 안되는 이유
상속받은 클래스로 인스턴스를 생성할 수 있고 잘못된 의도를 전달할 수 있습니다.

아래는 Spring이 제공하는 유틸리티 클래스입니다. 추상클래스로 만들었지만 아래처럼 상속을 받으면 인스턴스화가 가능합니다.

~~~java
// Spring Util Class
public abstract class PatternMatchUtils {
    public static boolean simpleMatch(@Nullable String[] patterns, String str) {
		if (patterns != null) {
			for (String pattern : patterns) {
				if (simpleMatch(pattern, str)) {
					return true;
				}
			}
		}
		return false;
	}
}

public class DefaultUtilityClass extends PatternMatchUtils {
  public static void main(String[] args) {
    DefaultUtilityClass utilityClass = new DefaultUtilityClass();
    utilityClass.simpleMatch("test", "test");
  }
}
~~~

두번째로 추상클래스로 만들면 '상속을 받으라는 의도인가?' 라는 생각이 들 수 있습니다. 따라서 위처럼 상속을 받은 후 인스턴스를 생성할 수 있는 위험이 있습니다.


## private 생성자를 사용하는 방법

위의 두가지 문제를 해결하기 위해서는 private 생성자와 AssertionError()을 사용해야 합니다. private 생성자를 사용해서 외부에서 인스턴스를 만드는 행위를 막고 AssertionError()를 사용해서 내부에서 인스턴스를 만드는 행위를 막을 수 있습니다. 주석으로 설명을 더해준다면 더욱 좋을 것 같습니다.

~~~java
public class UtilityClass {
    // 인스턴스를 만들면 안됩니다.
    private UtilityClass() {
        throw new AssertionError();
    }
}
~~~
