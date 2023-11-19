---
title:  "디자인 패턴 - 책임 연쇄"
categories:
  - Java
---

## 들어가며
오브젝트 책을 읽고 우테코 프리코스에 적용해보면서 애플리케이션(?) 설계에 재미가 들렸습니다. 예정되어 있는 BDD 프로젝트에서 확장성 있는 코드를 작성하기 위해서 GoF 디자인 패턴 강의를 들어오면 큰 도움이 될 것 같아서 강의를 수강했습니다. 

아래에서는 책임 연쇄 패턴이 필요한 이유를 알아보고 어떻게 코드를 작성해야 하는지 알아보겠습니다. 끝으로 책임 연쇄 패턴이 사용되고 있는 곳을 설명하고 글을 마치도록 하겠습니다.

## 책임 연쇄 패턴이 필요한 이유
책임 연쇄 패턴을 사용하지 않으면 크게 3가지의 문제점이 발생합니다.

1. 클라이언트가 구체적인 타입을 알아야 합니다.
2. 클라이언트 코드를 변경해야 합니다.
3. 2개 이상의 Handler를 사용하기 힘듭니다.

~~~java
public class Client {
    public static void main(String[] args) {
        Request request = new Request("무궁화 꽃이 피었습니다.");
        LoggingRequestHandler requestHandler = new LoggingRequestHandler();
        requestHandler.handler(request);
    }
}
~~~
~~~java
public class Request {
    private String body;
    public Request(String body) {
        this.body = body;
    }

    public String getBody() {
        return body;
    }
}
~~~
~~~java
public class LoggingRequestHandler extends RequestHandler{
    public void handler(Request request) {
        System.out.println("로깅");
        super.handler(request);
    }
}
~~~
~~~java
public class AuthRequestHandler extends RequestHandler {
    public void handler(Request request) {
        System.out.println("인증이 되었나?");
        System.out.println("이 핸들러를 사용할 수 있는 유저인가?");
        super.handler(request);
    }
}
~~~

위의 코드는 책임 연쇄 패턴으로 구현하지 않아 3가지의 문제점이 모두 발생합니다.

첫번째로 현재 Client 코드는 구체적인 RequestHandler의 타입을 알아야 합니다. 로깅을 원한다면 구체적인 LoggingRequestHandler 클래스를 알아야하며 인증을 원한다면 구체적인 AuthRequestHandler 클래스를 알고 있어야 하고 Client의 코드를 AuthRequestHandler 클래스로 변경해야 하고, 이는 자연스럽게 두번째 문제로 직결합니다.

그리고 마지막 문제로 로깅과 인증을 함께 사용할 수 없습니다.

## 책임 연쇄 패턴 구현
책임 연쇄 패턴은 메시지를 보내는 객체와 이를 받아 처리하는 객체들 간의 결합도를 없애기 위한 패턴입니다. 이 패턴을 구현하기 위해서는 추상화의 힘을 빌리고 후속 처리자를 연결해야 합니다.

Client는 구체적인 RequestHandler의 타입(LoggingRequestHandler, AuthReqeustHandler) 말고 RequestHandler라는 추상 클래스에 의존하도록 해야 합니다. 이렇게 한다면 Client가 구체적인 Handler 타입을 몰라도 되고, Client 코드를 변경하지 않아도 됩니다.

그리고 인증 절차와 로깅 기능, 추가 기능을 함께 사용하기 위해서 후속 처리자에 대한 정보를 저장하기 위해서 인스턴스 변수도 선언해야 합니다. 

~~~java
public class Client {

    private RequestHandler requestHandler; // 추상 클래스에 의존

    public Client(RequestHandler requestHandler) {
        this.requestHandler = requestHandler;
    }

    public void doWork() {
        Request request = new Request("이번 놀이는 뽑기입니다.");
        requestHandler.handle(request);
    }

    public static void main(String[] args) {
        AuthRequestHandler chain = new AuthRequestHandler(new LoggingRequestHandler(null));
        Client client = new Client(chain);
        client.doWork();
    }
}
~~~
~~~java
public abstract class RequestHandler {

    private RequestHandler next; // 후속 처리자를 위한 인스턴스 변수

    public RequestHandler(RequestHandler next) {
        this.next = next;
    }

    public void handle(Request request) {
        if (next != null) {
            next.handle(request);
        }
    }
}
~~~
~~~java
public class AuthRequestHandler extends RequestHandler {

    public AuthRequestHandler(RequestHandler nextHandler) {
        super(nextHandler);
    }

    @Override
    public void handle(Request request) {
        System.out.println("인증이 되었는가?");
        super.handle(request);
    }
}
~~~
~~~java
public class LoggingRequestHandler extends RequestHandler {

    public LoggingRequestHandler(RequestHandler next) {
        super(next);
    }

    @Override
    public void handle(Request request) {
        System.out.println("로깅");
        super.handle(request);
    }
}
~~~

## 책임 연쇄 패턴 적용 예시
책임 연쇄 패턴이 적용된 곳은 스프링 시큐리티에서 SecurityConfig, 자바의 서블릿 필터가 있습니다. 이와 관련된 내용은 기회가 된다면 포스팅해 보도록 하겠습니다.

## 결론
책임 연쇄 패턴을 사용하지 않으면 여러가지 문제가 발생할 수 있었습니다. 대부분의 문제는 추상화와 후속 처리자를 연결함으로써 해결할 수 있었습니다. 텍스트로는 이해하기가 어려워서 실제 적용해 보는 과정이 필요합니다. 우테코 프리코스 중 크리스마스 미션이 가장 적당한 문제입니다. 꼭 풀어보시면 좋을 것 같습니다~