---
published: true
title: "Effective Java Item 5 - 자원을 직접 명시하지 말고 의존 객체 주입을 사용하라"
categories:
  - Java
---

## 자원을 직접 명시할 경우 생길 수 있는 문제점
자원을 직접 명시할 경우 생기는 문제점은 아래의 두 가지 입니다.

1. 테스트 하기 어렵다.
2. 코드가 유연하지 않다. 

테스트할 때 모킹하기가 어렵습니다. 그리고 Dictionary에서 DefaultDictionary, KoreanDictionary 처럼 사용할 리소스가 변경될 경우 클라이언트 코드를 변경해야 합니다. 이는 OCP(확장에는 열려있고 변경에는 닫혀야 한다)를 위반합니다.

~~~java
public class SpellChecker {

    // 자원을 직접 명시한다 -> new DefaultDictionary(), new KoreanDictionary()
    private static final Dictionary dictionary = new Dictionary();

    // private 생성자로 외부에서 인스턴스 생성을 차단
    private SpellChecker() {}

    public static boolean isValid(String word) {
        return dictionary.contains(word);
    }

    public static List<String> suggestions(String typo) {
        return dictionary.closeWordsTo(typo);
    }
}
~~~

## 문제점 해결 - 의존 객체 주입 사용
인터페이스를 정의하고 외부에서 의존 객체를 주입하도록 코드를 변경하면 위의 두 가지 문제점을 해결할 수 있습니다. 테스트 할 때 모킹 객체를 주입해 테스트 시간을 단축할 수 있고 DefaultDictionary에서 KoreanDictionary로 변경할 때 SpellChecker 클래스의 코드를 변경하지 않아도 됩니다. 이는 OCP 원칙을 위반하지 않습니다.

~~~java
public interface Dictionary {

    boolean contains(String word);

    List<String> closeWordsTo(String typo);
}
~~~

~~~java
public class SpellChecker {

    private final Dictionary dictionary;

    private SpellChecker(Dictionary dictionary) {
        this.dictionary = dictionary;
    }

    public boolean isValid(String word) {
        return dictionary.contains(word);
    }

    public List<String> suggestions(String typo) {
        return dictionary.closeWordsTo(typo);
    }
}

~~~

의존성이 수천 개나 되는 큰 프로젝트에서는 코드를 어지럽게 만들기도 합니다. 하지만 대표적으로 스프링 같은 의존 객체 주입 프레임워크를 사용하면 손쉽게 해결할 수 있습니다.

## 마치며
이펙티브 자바의 "자원을 직접 명시하지 말고 의존 객체 주입을 사용하라" 챕터를 살펴봤습니다. 끝에 스프링 프레임워크에 대해서 언급을 했는데 다음 포스트는 스프링 IoC에 대해서 살펴보겠습니다.