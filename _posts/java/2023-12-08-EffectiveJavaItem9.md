---
published: true
title:  "Effective Java Item 9 - try-finally 보다는 try-with-resources를 사용하라"
categories:
  - Java
---

## 서론

전공 수업인 플랫폼기반프로그래밍(JAVA) 강의를 듣고 과제를 하던 중 PPT에 나오는 try-catch-finally 코드가 지저분해 보였습니다. Effective Java에서 try-finally를 쓰지말고 try-with-resources를 사용하라는 챕터가 있었던걸 기억하고 공부해 봤습니다.

try-with-resources를 사용하면 발생하는 장점 3가지와 어떻게 과제에 적용했는지 알아보고 글을 마치도록 하겠습니다.


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


## 리소스 Leak을 방지할 수 있고 코드가 깔끔해진다.

아래의 코드는 이번 과제에서 작성한 코드와 비슷한데 리소스를 Leak할 수 있는 코드입니다. finally 코드 블럭 안에서 in.close() 메서드에 문제가 생긴다면 out.close() 메서드는 실행 "시도" 조차도 하지 않습니다.

~~~java
public class OtherTopLine {

    private static final int BUFFER_SIZE = 8 * 1024;

    public static void main(String[] args) throws IOException {
        String src = args[0];
        String dst = args[1];
        copy(src, dst);
    }

    private static void copy(String src, String dst) throws IOException {
        InputStream in = new FileInputStream(src);
        OutputStream out = new FileOutputStream(dst);

        try {
            byte[] buf = new byte[BUFFER_SIZE];
            int n;
            while ((n = in.read(buf)) >= 0) {
                out.write(buf, 0, n);
            }
        } finally {
            in.close();
            out.close();
        }
    }
}
~~~

위의 문제점은 try로 한번 더 감싸서 리소스 낭비 문제를 해결할 수 있습니다. 이렇게 한다면 out.close()에서 문제가 발생해도 in.close() 메서드를 실행하려는 "시도"를 할 수 있습니다.

~~~java
public class OtherTopLine {
    
    private static final int BUFFER_SIZE = 8 * 1024;

    public static void main(String[] args) throws IOException {
        String src = args[0];
        String dst = args[1];
        copy(src, dst);
    }

    private static void copy(String src, String dst) throws IOException {

        InputStream in = new FileInputStream(src);
        try {
            OutputStream out = new FileOutputStream(dst);
            try {
                byte[] buf = new byte[BUFFER_SIZE];
                int n;
                while ((n = in.read(buf)) >= 0) {
                    out.write(buf, 0, n);
                }
            } finally {
                out.close();
            }
        } finally {
            in.close();
        }
    }
}
~~~

위의 지저분한 코드를 try-with-resources를 사용하면 훨씬 간결해지고 자동으로 이중 try 처럼 close를 해주것을 보장하기 때문에 리소스 Leak도 생기지 않습니다.

~~~java
public class OtherTopLine {

    private static final int BUFFER_SIZE = 8 * 1024;

    public static void main(String[] args) throws IOException {
        String src = args[0];
        String dst = args[1];
        copy(src, dst);
    }

    private static void copy(String src, String dst) throws IOException {
        try (InputStream in = new FileInputStream(src);
             OutputStream out = new FileOutputStream(dst)) {
            byte[] buf = new byte[BUFFER_SIZE];
            int n;
            while ((n = in.read(buf)) >= 0) {
                out.write(buf, 0, n);
            }
        }
    }
}
~~~

## 결론

무조건 try-with-resources를 사용해야 합니다. 보통 트레이드 오프가 발생하기 마련인데 try-with-resources는 장점밖에 없습니다. 이펙티브 자바의 저자가 한 말씀을 적으면서 글을 마무리 하도록 하겠습니다.

> 꼭 회수해야 하는 자원을 다룰 때는 try-finally 말고, try-with-resources를 사용하자. 예외는 없다. 코드는 더 짧고 분명해지고, 만들어지는 예외 정보도 훨씬 유용하다. try-finally로 작성하면 실용적이지 못할 만큼 코드가 지저분해지는 경우라도, try-with-resources로는 정확하고 쉽게 자원을 회수할 수 있다. 
