---
published: true
title:  "Effective Java Item 9 - try-finally 보다는 try-with-resources를 사용하라"
categories:
  - Java
---

## 서론

전공 수업인 플랫폼기반프로그래밍(JAVA) 강의를 듣고 과제를 하던 중 PPT에 나오는 try-catch-finally 코드가 지저분해 보였습니다. Effective Java에서 try-finally를 쓰지말고 try-with-resources를 사용하라는 챕터가 있었던걸 기억하고 공부해 봤습니다.

try-with-resources를 사용하면 발생하는 장점 3가지에 대해서 알아보고 어떻게 과제에 적용했는지 알아보고 글을 마치도록 하겠습니다.


## try-with-resources는 stackTrace가 가능하다.

try-with-resources는 try-finally와는 다르게 stackTrace에 발생한 모든 예외가 잡힙니다. 반대로 말하면 try-finally는 모든 예외가 잡히지 않아 디버깅을 어렵게 만듭니다.

~~~java
public class BadBufferedReader extends BufferedReader {

    public BadBufferedReader(Reader in, int sz) {
        super(in, sz);
    }

    public BadBufferedReader(Reader in) {
        super(in);
    }

    @Override
    public String readLine() throws IOException {
        throw new CharConversionException();
    }

    @Override
    public void close() throws IOException {
        throw new StreamCorruptedException();
    }
}
~~~
~~~java
public class TopLine {

    public static void main(String[] args) throws IOException {
        System.out.println(firstLineOfFile("pom.xml"));
    }

    private static String firstLineOfFile(String path) throws IOException {
        BufferedReader br = new BadBufferedReader(new FileReader(path));
        try {
            return br.readLine();
        } finally {
            br.close();
        }
    }
}
~~~

![](https://github.com/02ggang9/02ggang9.github.io/blob/master/_posts/images/java/effectiveJava/item9/exception1.png?raw=true)

위의 실행 결과를 살펴보면 알 수 있듯이 try-finally는 발생한 모든 예외를 잡아주지 못합니다. readLine() 메서드를 실행했을 때는 CharConversionException 예외를 발생하고 close() 메서드를 실행했을 때는 StreamCorruptedException 예외를 발생하도록 오버라이딩을 했지만 결과는 "가장 최근에 발생한 예외"인 StreamCorruptedException 예외만 잡아줍니다.

예외는 처음 발생한 시작점이 가장 중요한데 try-finally는 이를 잡아주지 못하는 치명적인 단점이 있다는 것을 확인할 수 있습니다.

~~~java
public class TopLineWithResources {

    public static void main(String[] args) throws IOException {
        System.out.println(firstLineOfFile("README.md"));
    }

    private static String firstLineOfFile(String path) throws IOException {
        try (BufferedReader br = new BadBufferedReader(new FileReader(path))) {
            return br.readLine();
        }
    }
}
~~~

![](https://github.com/02ggang9/02ggang9.github.io/blob/master/_posts/images/java/effectiveJava/item9/exception2.png?raw=true)

반면에 try-with-resources를 사용하면 발생한 모든 예외를 잡아주는 것을 확인할 수 있습니다. 이는 try-with-resources의 바이트코드를 살펴보면 이유를 알 수 있습니다.

![](https://github.com/02ggang9/02ggang9.github.io/blob/master/_posts/images/java/effectiveJava/item9/exception3.png?raw=true)

br.close()을 실행하고 발생한 예외를 catch 해서 suppress 하는 행위를 확인할 수 있습니다. 


## 메모리 Leak을 방지할 수 있다.




## 코드가 깔끔해진다.


## 결론