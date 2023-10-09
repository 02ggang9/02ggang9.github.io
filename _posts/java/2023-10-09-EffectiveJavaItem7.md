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


### 참조를 담은 변수를 유효범위 밖으로 밀어낸다.
참조를 담은 변수를 유효범위 밖으로 밀어내는 방법은 아래에서 설명하겠습니다.

## 마치며
