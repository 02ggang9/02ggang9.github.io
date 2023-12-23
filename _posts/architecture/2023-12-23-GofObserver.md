---
published: true
title:  "GoF Design Pattern - Observer Pattern"
categories:
  - architecture
---

## 요약

옵저버 패턴은 Observer가 자동으로 객체의 상태를 확인하고 행동하게 할 수 있습니다. 불필요한 리소스를 줄이고 객체 간의 관계를 디커필링할 수 있습니다. Spring Reactive의 코어 기술의 근간인 만큼 알아두면 좋을 것 같습니다.

## 서론

KEEPER R2 프로젝트를 할 때 미니언 선배님께서 옵저버 패턴을 도입해 보면 어떠냐는 리뷰를 해 주셨습니다. 4개월 전 얘기라 자세한 PR을 찾지 못했지만, 기억이 나서 한번 공부해 봤습니다. 아래에서는 Observer 패턴이 나온 이유와 구현 방법, 장단점에 대해서 알아보도록 하고 글을 마치도록 하겠습니다.

## Observer 패턴은 무엇이고 왜 만들어졌나?

Observer 패턴은 상태의 변경을 지켜보다가 상태가 변경될 때 그 변화를 통지받고 자동으로 갱신해 주는 패턴입니다. 이 패턴이 나온 이유는 아래의 코드를 보면 알 수 있습니다. 

~~~java
// Subject
public class ChatServer {

    private Map<String, List<String>> messages;

    public ChatServer() {
        this.messages = new HashMap<>();
    }

    // 옵저버를 추가해주는 메서드
    public void add(String subject, String message) {
        if (this.messages.containsKey(subject)) {
            this.messages.get(subject).add(message);
        } else {
            List<String> messageList = new ArrayList<>();
            messageList.add(message);
            this.messages.put(subject, messageList);
        }
    }

    public List<String> getMessage(String subject) {
        return this.messages.get(subject);
    }

}
~~~

~~~java
// Observer
public class User {

    private ChatServer chatServer;

    public User(ChatServer chatServer) {
        this.chatServer = chatServer;
    }

    public void sendMessage(String subject, String message) {
        chatServer.add(subject, message);
    }

    public List<String> getMessage(String subject) {
        return this.chatServer.getMessage(subject);
    }
}
~~~

~~~java
public class Client {

    public static void main(String[] args) {
        ChatServer chatServer = new ChatServer();

        User user1 = new User(chatServer);
        user1.sendMessage("디자인패턴", "이번엔 옵저버 패턴입니다.");
        user1.sendMessage("롤드컵2021", "LCK 화이팅!");

        User user2 = new User(chatServer);
        System.out.println(user2.getMessage("디자인패턴"));

        user1.sendMessage("디자인패턴", "예제 코드 보는 중..");
        System.out.println(user2.getMessage("디자인패턴"));
    }
}
~~~

위의 코드에서는 User가 메시지를 서버에게 보내고 나를 포함한 다른 User가 보낸 메시지를 getter를 통해서 가져오는 방식임을 알 수 있습니다.

그런데 만약 어떠한 user도 메시지를 send 하지 않았는데 매번 getter를 사용해서 메시지를 확인해야 한다면 리소스 낭비가 심할 것입니다. 이러한 문제점을 해결하기 위해서 자동으로 객체의 상태를 체크하고 업데이트하는 Observer 패턴이 나왔습니다.

## Observer 패턴 구현

~~~java
public interface Subscriber {
    void handleMessage(String message);
}
~~~

~~~java
public class ChatServer {

    // Memory Leak 발생 가능 지점
    private Map<String, List<Subscriber>> subscribers = new HashMap<>();

    // Observer 추가
    public void register(String subject, Subscriber subscriber) {
        if (this.subscribers.containsKey(subject)) {
            this.subscribers.get(subject).add(subscriber);
        } else {
            List<Subscriber> list = new ArrayList<>();
            list.add(subscriber);
            this.subscribers.put(subject, list);
        }
    }

    // Observer 삭제
    public void unregister(String subject, Subscriber subscriber) {
        if (this.subscribers.containsKey(subject)) {
            this.subscribers.get(subject).remove(subscriber);
        }
    }

    // 등록된 Observer 에게 Notify
    public void sendMessage(User user, String subject, String message) {
        if (this.subscribers.containsKey(subject)) {
            String userMessage = user.getName() + ": " + message;
            this.subscribers.get(subject).forEach(s -> s.handleMessage(userMessage));
        }
    }
}
~~~

~~~java
public class User implements Subscriber {

    private String name;

    public User(String name) {
        this.name = name;
    }

    public String getName() {
        return name;
    }

    // Subject가 Nofity 해주면 아래의 메서드가 실행 -> Update 부분
    @Override
    public void handleMessage(String message) {
        System.out.println(message);
    }
}
~~~

~~~java
public class Client {

    public static void main(String[] args) {
        ChatServer chatServer = new ChatServer();
        User user1 = new User("keesun");
        User user2 = new User("whiteship");

        chatServer.register("오징어게임", user1);
        chatServer.register("오징어게임", user2);

        chatServer.register("디자인패턴", user1);

        chatServer.sendMessage(user1, "오징어게임", "일남이야..");
        chatServer.sendMessage(user2, "디자인패턴", "옵저버 패턴으로 만든 채팅");

        chatServer.unregister("디자인패턴", user2);

        chatServer.sendMessage(user2, "디자인패턴", "옵저버 패턴 창, 단점 보는 중");
    }
}
~~~

Client 코드를 보면 chatServer(Subject)가 User(Observer)를 추가(register)하고 삭제(unregister)도 하는 것을 확인할 수 있습니다. 또, chatServer(subject)의 sendMessage(Notify)를 통해 등록된 모든 옵저버를 순회하면서 Update를 수행합니다.

이렇게 옵저버 패턴을 적용한다면 스케줄링 또는 다른 방법으로 매번 getter를 써가면서 객체의 상태를 가져오지 않아도 됩니다. 만약 객체의 상태가 오랫동안 변경되지 않는다면 리소스 낭비가 심해질 것입니다.

## 장단점

### 장점

상태를 변경하는 객체와 변경을 감지하는 객체의 관계를 느슨하게 유지할 수 있습니다.

Subject의 상태 변경을 주기적으로 조회하지 않고 자동으로 감지할 수 있습니다.

런타임에 옵저버를 추가하거나 제거할 수 있습니다.

### 단점

복잡도가 증가합니다

다수의 Observer 객체를 등록 이후 해지하지 않는다면 memory Leak이 발생할 수 있습니다.

## 결론

어떤 객체의 상태를 바라보다가 변경이 일어나는 순간 특정 행동을 취하고 싶다면 Observer 패턴을 적극 이용합시다. Observer 패턴을 사용하면 객체 간의 커플링을 제거할 수 있고, Subject의 상태 변경을 주기적으로 조회하지 않고 자동으로 감지하기 때문에 리소스 낭비가 발생하지 않습니다. 단점은 복잡도가 증가하고 memory Leak이 발생할 수 있습니다.

Observer 패턴은 Spring Reactive 의 핵심 코어에서 사용되고 있으므로 옵저버 패턴을 잘 알고 있으면 도움이 많이 됩니다. 다음 포스팅은 토비님의 라이브 코딩 영상을 보면서 다시한번 옵저버 패턴에 대해서 알아보도록 하겠습니다.

