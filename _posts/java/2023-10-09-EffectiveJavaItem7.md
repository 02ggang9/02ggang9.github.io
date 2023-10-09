---
published: false
title: "Effective Java Item 7 - 다 쓴 객체 참조를 해제하라."
categories:
  - Java
---

이번 Item은 "다 쓴 객체 참조를 해제하라" 주제로 주로 메모리 누수에 관한 내용을 다루고 있습니다. 보이지 않는 메모리 누수가 어디에서 발생할 수 있는지 알아보고 해결 방법에 대해서 알아보도록 하겠습니다.

## 메모리 누수

### 스택
스택이 다 쓴 참조(obsolete reference)를 가지고 있는 경우 스택에서 꺼내진 객체들을 GC가 회수 하지 않습니다. 예시로 아래의 코드에서 OutOfMemoryError가 발생할 수 있습니다.

~~~java
public class Stack {

    private Object[] elements;
    private int size = 0;
    private static final int DEFAULT_INITIAL_CAPACITY = 16;

    public Stack() {
        elements = new Object[DEFAULT_INITIAL_CAPACITY];
    }

    public void push(Object e) {
        ensureCapacity();
        elements[size++] = e;
    }

    public Object pop() { // 넣기만 한다. 해제를 안함.
        if (size == 0) {
            throw new EmptyStackException();
        }
        return elements[--size];
    }

    private void ensureCapacity() {
        if (elements.length == size) {
            elements = Arrays.copyOf(elements, 2 * size + 1);
        }
    }

}
~~~

### 캐시
캐시 또한 메모리 누수를 일으키는 주범입니다. 캐시에 객체 참조를 넣고 나서 객체를 다 쓴 뒤로도 놔두는 일이 빈번합니다. 아래의 코드는 HashMap을 이용한 캐싱 예시입니다. GC가 일어나도 HashMap의 Element는 삭제되지 않습니다.

~~~java
public class PostRepository {

    private Map<CacheKey, Post> cache;

    public PostRepository() {
        this.cache = new HashMap<>();
    }

    public Post getPostById(Integer id) {
        CacheKey key = new CacheKey(id);
        if (cache.containsKey(key)) {
            return cache.get(key);
        } else {
            // TODO DB에서 읽어오거나 REST API를 통해 읽어올 수 있습니다.
            Post post = new Post();
            cache.put(key, post);
            return post;
        }
    }

    public Map<CacheKey, Post> getCache() {
        return cache;
    }
}
~~~

~~~java
public class PostRepositoryTest {

    public static void main(String[] args) throws InterruptedException {
        PostRepository postRepository = new PostRepository();
        Integer p1 = 1;
        Post postById = postRepository.getPostById(p1);

        System.out.println(postRepository.getCache().isEmpty()); // false

        p1 = null;
        // TODO run gc
        System.out.println("run gc");
        System.gc();
        System.out.println("wait");
        Thread.sleep(3000L);

        System.out.println(postRepository.getCache().isEmpty()); // false
    }
}
~~~

## 해결 방법

### 해당 참조를 다 썻을 때 null 처리
스택에서 생겨나는 메모리 누수는 해당 참조를 다 썻을 때 null 처리를 함으로써 해결할 수 있습니다. 책에서는 모든 객체를 다 쓰자마자 일일이 null 처리를 하면 코드가 필요 이상으로 지저분해지기 때문에 혈안이 되지 말자고 말합니다.

~~~java
    public Object pop() {
        if (size == 0) {
            throw new EmptyStackException();
        }

        Object result = elements[--size];
        elements[size] = null; // 다 쓴 참조 해제
        return result;
    }
~~~

### WeakHaskMap 사용
WeakHashMap을 사용해서 해결하는 방법은 아래에서 설명하겠습니다.



### 참조를 담은 변수를 유효범위 밖으로 밀어낸다. (ITEM 57) -> 따로 빼기



























weakHashMap -> weak 래퍼런스를 key 로 가진다.
strong, soft, weak, pentom

key가 더이상 참조되지 않으면 그 키를 가지고 있는 value를 맵에서 뺀다. strong 한 래퍼런스가 없으면 가비지 컬렉션 한테 날아간다.

---
NullPointException

널 포인트 문제를 확인하는 방법과 처리하는 방법에 대해서 알아본다. 그리고 NE를 없앨 수 있는? 완화할 수 있는 방법에 대해서도 알아본다

null을 리턴 하는 것이 아니라 에러를 던지거나 그런 자바 8에서는 Optional이라는 것이 생겼다.

return null; -> Optional.empty();
return Optional.of(new MemberShip());

~~~java

Channel channel = new Channel();
Optional<MemberShip> memberShip = channel.defaultMemberShip();

memberShip.hello(); -> 이렇게는 당연히 안됨. 왜냐하면 Optional 클래스에 hello 메서드가 있는 것이 아니기 때문에 ㅋㅋ

Consumer -> 뭔가 하나를 받고 리턴을 하는 건 없는 친구임 (void 리턴)
m -> 이 오브젝트 한 개를 받는 것이다.
memberShip.ifPresent(MemberShip::hello);

MemberShip memberShip = optional.get(); // 비어있는 것에서 꺼낼려고 하면 No value present 라는 에러 메시지를 던져준다.

Optional을 리턴타입으로만 거의 써야함. 매개변수에 집어넣으면 안됨 ㅋㅋ;

왜냐하면 의미가 없기 때문에. 파라미터가 옵셔널인지 또 검사를 해야함.

Optional<Set>, <List> 같은걸 감싸지 마라. Set이랑 List는 null인지 확인하는 메서드가 이미 있다!

primitive 타입은 못 감싼다. 따라서 OptionalInger 같은 걸 써라!


~~~

---
WeakHashMap || weak reference

WEAK -> 더 이상 사용하지 않는 객체르 GC가 일어날 때 자동으로 없애준다.

List, Map 등은 GC가 일어난다고 자동으로 비워주지 않음. 키가 더이상 강하게 레퍼런스 되는 곳이 없다면 weak 레퍼런스가 된다면 자동으로 제거가 된다.

벨류를 보존하는데 조금 더 집중해야 한다. 벨류가 유효한 경우 키도 유효해야 한다. 그래서 위크해쉬맵을 잘 안씀.

근데 반대로 키가 유효하지 않으면 밸류가 무의미 해 지는 경우 위크해쉬 맵 추천!

Key를 래퍼런스로 만드는 경우, 


String, primitive 타입을 쓰는 경우 지워지지 않음. 스트롱 래퍼런스가 남아있다고 생각함.

Key를 Integer로 받도록 만들면 캐쉬가 삭제되지 않음. 1이라는 값, 자주 사용되는 primitive 타입은 JVM 내부에 이미 캐쉬되어 있음 상수 pool 같은 곳에.. Map의 키 값은 primitive 타입을 못 쓰고 래퍼 타입으로 써야 함.

그래서 weakHashMap을 쓰는 경우 CachKey 같은 래퍼런스를 만들어서 써라! 하나 감싸는 걸 만들어서 쓰면 원하는대로 동작한다..!!

~~~java

List<WeakReference<User>> users;이렇게 쓰면 안된다!

reference 
new 를 하건 팩토리 메서드를 쓰건 빌더를 쓰건 이렇게 = 으로 할당하는 걸 stringReference 라고 한다. 스트롱 레퍼런스가 유효하지 않으려면 

public List<WeakReference<User>> getUsers() {
    // 가비지 털렉션 대상이 된다.
    ChatRoom localRoom = new ChatRoom();
    localRoom = null;
    return users;
}

~~~

softReference

~~~java

SoftReference<Object> soft = new SoftReference<>(Strong);

// Soft -> Object + Sring -> Object

어떤 오브젝트를 스트롱 레퍼런스 하는 애들은 없고 soft 레퍼만 하면 메모리가 필요한 상황에만 (정말 치워야겠다. 뭘 놓을 공간이 없다) 그런 경우에 GC 하면서 치운다. 그때 소프트 레퍼런스가 없어진다. 근데 사실 잘 안 없어진다.

weak 레퍼런스는 진짜 약하게 연결이 되어있는거임.

WeakReference<Object> soft = new WeakReference<>(Strong);

얘는 GC 일어나면 걍 없어짐 ㅋㅋ

PhantomReference (유령 레퍼런스)
얘는 그냥 팬텀이 남는다. String 인스턴스 대신에 남아있는거임. 

ReferenceQueue<Object> phantom = new ReferenceQueue<>(Strong, rq); //reference quque를 넘겨줘야 함

팬텀 래퍼런스(phantom)을 래퍼런스 큐에 넣어준다.

1. 자원 정리할 때 쓸 수 있음. 파이널라이저 보다는 그래도 조금 더 나은 리소스 정리하는 방법임

파이널라이저 < 팬텀 < 

2. 언제 메모리가 해제되는지 사라지는지를 알 수 있음. 왜냐하면 사라짐과 동시에 큐에 들어가기 때문에
큐에 들어갔는지 확인을 하고 어 잇네? -> 큰 오브젝트가 사라졌구나! -> 다른 행동을 하도록 로직 작성 가능

언제 없어지는 애매모호한 경우라서 그냥 안 쓰는게 좋ㄱ겠다! 권장ㅎ지 ㅇ낳음





~~~


---
ShecduledThreadPoolExcutor

Thread, Runnable, ExcutorService

스레드를 만드는 것은 시스템 리로스를 굉장히 많이 쓰는거임.

근데 스레드 100개를 안말들어도 비동기로 별로의 서비스로 해결할 수 있다면? 그게 더 ㅈ호은거지

~~~java

ExecutorServuce service = Executors.newFixedThreadPool(10);

그래서 스레드 10개 만 가지고 100개의 작업을 한다. 그래서 스레드 풀의 개수를 신경써야 한다..

신경 쓸거는

CPU에 집중적인 작업이냐 , IO에 집중적인 작업이냐!

CPU 개수 만큼만 만들게 될 것이다! CPU 개수를 구하는 방법은 

int numberOfCpu = Runtime.getRunTime().availableProcessors();
ExecutorServuce service = Executors.newFixedThreadPool(numberOfCpu);
blocking queue -> 컨커런트하게 접근 가능

IO 인텐시브한 작업. HTTP Call을 해서 값을 가져오거나 DB에서 값을 가져오거나. IO 때문에 4개를 다 써버리면 IO 기다리느라 아무것도 못하는겅미





newCachedThreadPool() 
놀고있는 쓰레드가 있으면 사용하고 없으면 새로 만듬. 너무 놀고 있는 60초 동안 놀고 있는 스레드는 없어짐


ScheduledThreadPool 

순차적으로 작업이 들어온다고 해서 순차적으로 하는게 아니라 스케줄을 보고 순서를 정한다.


Runnable 은 리턴이 없고 그냥 작업을 하고 끝내는 거임.
근데 리턴 값을 받고 싶으면 어떡함? -> 그래서 생긴게 콜러블 Callable임. 리턴으로 Future 타입을 받을 수 있음.

~~~

