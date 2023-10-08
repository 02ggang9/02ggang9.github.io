---
published: true
title: "Effective Java Item 6 - 불필요한 객체 생성을 피하라"
categories:
  - Java
---

## String 객체 사용 주의점
new String()으로 객체를 생성하는 방법은 사용하면 안됩니다. 왜냐하면 new 연산자로 String 객체를 생성할 경우 String은 Heap 영역 내 "String Pool"에 저장되지 않고 매번 새로운 객체를 생성하기 때문입니다. String literal로 String 객체를 생성하면 "String Pool"에 해당 문자열이 있는지 확인하고 있으면 기존 값을 참조하도록 설정하고 없으면 새로 만든 후 참조합니다.

~~~java
public class StringTest {
    public static void main(String[] args) {
        String str1 = new String("ggang9");
        String str2 = "ggang9";
        String str3 = "ggang9";

        System.out.println(str1 == str2); // false
        System.out.println(str2 == str3); // true
    }
}
~~~

## 값비싼 객체는 재사용해 성능을 개선한다.
값비싼 객체는 재사용해 성능을 개선시킬 수 있습니다. 내가 만든 객체가 '비싼 객체' 인지는 매번 정확히 알 수 없습니다. 따라서 성능 측정을 통해서 지표를 확인한 후 판단하시면 될 것 같습니다.

이펙티브 자바 책에서는 아래 처럼 생성 비용이 비싼 정규 표현식을 예로 들었습니다. isRomanNumeralSlow처럼 인스턴스를 생성해버리면 비싼 객체를 생성하고 바로 GC의 대상이 되기 때문에 성능이 좋지 않습니다. ROMAN 처럼 한 인스턴스로 만들어 사용한다면 성능을 개선할 수 있습니다.

~~~java
public class Refex {

    static boolean isRomanNumeralSlow(String s) {
        return s.matches("^(?=.)M*(C[MD]|D?C{0,3})(X[CL]|L?X{0,3})(I[XV]|V?I{0,3})$");
    }

    private static final Pattern ROMAN = Pattern.compile("^(?=.)M*(C[MD]|D?C{0,3})(X[CL]|L?X{0,3})(I[XV]|V?I{0,3})$");

    static boolean isRomanNumeralFast(String s) {
        return ROMAN.matcher(s).matches();
    }

    public static void main(String[] args) {
        boolean result = false;
        long start = System.nanoTime();
        for (int i = 0; i < 100; i++) {
            result = isRomanNumeralSlow("MCMLXXVI"); // 3925083
            result = isRomanNumeralFast("MCMLXXVI"); // 608291
        }
        long end = System.nanoTime();

        System.out.println(end - start);
        System.out.println(result);
    }
}
~~~

## 오토 박싱
오토 박싱에 관한 내용은 아래의 포스팅에서 정리했습니다.

[EffectiveJavaItem61](https://02ggang9.github.io/java/EffectiveJavaItem61/)
