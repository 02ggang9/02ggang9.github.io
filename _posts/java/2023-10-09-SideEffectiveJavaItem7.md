---
published: false
title: "Side Effective Java Item 7 - WeakHashMap과 Optional"
categories:
  - Java
---

## WeakHashMap 
WeakHashMap은 WeakReference의 특성을 이용해 HashMap의 Element를 자동으로 제거합니다. 아래의 PostRepositoryTest 를 보시면 객체 참조가 null이 되고 GC가 일어나면 자동으로 WeakMapHash의 Element가 삭제되는 것을 확인할 수 있습니다. 

~~~java
public class CacheKey {
    private Integer value;
    private LocalDateTime created;

    public CacheKey(Integer value) {
        this.value = value;
        this.created = LocalDateTime.now();
    }
}
~~~

~~~java
public class PostRepository {
    private Map<CacheKey, Post> cache;

    public PostRepository() {
        this.cache = new WeakHashMap<>();
    }

    public Post getPostById(CacheKey key) {
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
        CacheKey key = new CacheKey(1);
        postRepository.getPostById(key);

        System.out.println(postRepository.getCache().isEmpty()); // false

        key = null;
        // TODO run gc
        System.out.println("run gc");
        System.gc();
        System.out.println("wait");
        Thread.sleep(3000L);

        System.out.println(postRepository.getCache().isEmpty()); // true
    }
}
~~~

### 주의점
String, Integer 타입 등이 Key 값이라면 null이 되어도 GC 시점에 사라지지 않습니다.왜냐하면 자주 사용되는 String과 Integer 값들은 캐싱되기 때문입니다. (String pool, Constant pool 등) 따라서 래퍼런스 타입으로 감싸서 Key 값으로 사용하는 방법을 사용해야 합니다.

### StrongReference
StringReference는 평소 저희가 사용하는 가장 일반적인 참조 유형입니다. Strong Reference가 있는 객체는 GC 대상이 되지 않습니다.

~~~java
Integer a = 1;
~~~

### SoftReference
SoftReference는 null이 되면 GC의 대상이 됩니다. 하지만 GC가 일어난다고 해서 사라지지 않습니다. 메모리가 부족한 경우에만 GC으로 사라지게 됩니다.

~~~java
// 생성 방법
SoftReference<Integer> soft = new SoftReference<Integer>(a);
~~~

### WeakReference
WeakReference는 null이 되면 GC의 대상이 됩니다. SoftReference와 다르게 GC가 발생하는 시점에 즉시 사라지게 됩니다.

~~~java
WeakReference<Integer> weak = new WeakReference<Integer>(a);
~~~



## NullPointException

### Optional