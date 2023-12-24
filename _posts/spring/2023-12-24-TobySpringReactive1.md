---
published: false
title:  "Spring - Toby Spring Reactive Streams (1) Observer 패턴"
categories:
  - spring
---

## 요약

옵저버 패턴을 사용하면 비동기로 동작하는 코드를 쉽게 작성할 수 있고 스케일링이 가능합니다.

## 서론


저번에 백기선님의 GoF Design Pattern - Observer 패턴을 공부하고 포스팅 했습니다. 이번에는 토비님의 [유튜브 라이브 코딩 방송](https://www.youtube.com/watch?v=8fenTR3KOJo&list=PLv-xDnFD-nnmof-yoZQN8Fs2kVljIuFyC&index=10)을 보면서 따라 쳐봤고, 내용을 정리해봤습니다. 영상 빌드업이 굉장히 탄탄했고 이 글을 보시는 다른 분들도 꼭 한번쯤 보셨으면 좋겠습니다. 아래에서는 Pull 방식과 Push 방식의 차이점에 대해서 알아보고, JAVA 9 버전에 Deprecated 된 Observer 인터페이스를 사용해 옵저버 패턴을 만들어 보고, 똑같이 스프링 진영에서 제공하는 Publiser, Subscriber 인터페이스를 이용해 옵저버 패턴을 만들어 보겠습니다.


## for-each와 iterator는 Pull 방식이다.

Pull 방식은 데이터를 달라고 요청하는 방식입니다. Pull 방식으로 데이터를 가져오는 대표적인 방법은 forEach와 iterator를 사용하는 것입니다. 아래의 코드에서 forEach와 iterator를 사용해서 데이터를 Pull 해서 가져오고 있습니다.

영상의 초반부에서 Iterable 인터페이스에 대해서 설명을 해 주셨습니다. Iterable 인터페이스에 대한 설명은 아래의 주석에서 확인하실 수 있는데 Iterable을 구현하는 모든 오브젝트는 for-each가 가능하다고 합니다. 컬렉션이 아니더라도 Iterable 인터페이스만 구현하면 for-each문을 사용할 수 있습니다.

~~~java
@SuppressWarnings("deprecation")
public class Ob {

    public static void main(String[] args) {

        /*
         * List 는 Iterable 의 하위 타입입니다.
         * Implementing this interface allows an object to be the target of the enhanced for statement (sometimes called the "for-each loop" statement)
         * 위의 말은 Iterable 을 구현하는 모든 오브젝트는 for-each 가 가능하다는 말입니다.
         * 컬렉션이 아니더라도 원소 하나씩 꺼내서 순회할 수 있다는 것입니다.
         * */

        List<Integer> list = Arrays.asList(1, 2, 3, 4, 5);
        for (Integer i : list) {
            System.out.println(i);
        }

        Iterable<Integer> iter = Arrays.asList(1, 2, 3, 4, 5);
        for (Integer i : iter) {
            System.out.println(i);
        }

        /*
         * Iterable 은 구현해야 할 메서드가 1개 밖에 없기 때문에 람다로 가능합니다.
         * 하지만 Iterator 는 구현해야 할 메서드가 많기 때문에 람다로는 불가능합니다.
         * */

        Iterable<Integer> iter3 = () ->
                new Iterator<>() {

                    int i = 0;
                    final static int MAX = 10;

                    @Override
                    public boolean hasNext() {
                        return i < MAX;
                    }

                    @Override
                    public Integer next() {
                        return ++i;
                    }
                };

        for (Integer i : iter3) {
            System.out.println(i); // output : 1~10
        }

        for (Iterator<Integer> it = iter3.iterator(); it.hasNext(); ) { // 옛날 방식의 for문
            System.out.println(it.next()); // output : 1~10
        }
    }
}
~~~


## Push 방식의 Observer Pattern (JAVA)

Push 방식은 데이터를 밀어주는 방식입니다. Push 방식으로 데이터를 밀어주는 대표적인 방법은 Observer 패턴을 사용하는 것입니다. 아래의 코드에서는 JAVA 9 버전부터 Deprecated 된 Observer 인터페이스를 사용해 옵저버 패턴을 구현했습니다. 옵저버 패턴에 관한 내용은 [OberverPattern](https://02ggang9.github.io/architecture/GofObserver/)에서 확인하실 수 있습니다. 


~~~java
@SuppressWarnings("deprecation")
public class Ob {

    public static void main(String[] args) {

        /*
         * Iterable <---> Observable (쌍대성 = 궁극적인 기능은 똑같은데 표현이 반대입니다.)
         * Pull           Push (데이터를 멀어주는 방식)
         * 문을 잡아 땡겨서 연다
         * 문을 밀어서 연다
         * */

        /*
         * Source -> Event/Data -> Observer
         * Update() 메서드 -> notifyObservers(i)가 호출되면 현재 옵저버블에 등록되어 있는 옵저버에게 전달됩니다.
         * Pull은 정보를 줘 -> return 으로 값을 줘야 합니다.
         * Push는 데이터를 밀어 넣는 것 -> 따라서 return 값이 없습니다. void
         * public DATA method() <--> public void method(DATA)
         * */

        Observer ob = new Observer() {
            @Override
            public void update(Observable o, Object arg) {
                System.out.println(Thread.currentThread().getName() + " " + arg);
            }
        };

        System.out.println("================");

        IntObservable io = new IntObservable();
        io.addObserver(ob);
//        io.run();

        /*
         * ExecutorService es = Executors.newSingleThreadExecutor(); 를 먼저 했는데
         * MAIN EXIT가 먼저 끝났습니다.
         * 별개의 쓰레드에서 동작을 하는 코드를 쉽게 작성할 수 있습니다(Push). 반면에 이터레이터를 사용하면 너무 어려워 집니다(Pull).
         * */

        ExecutorService es = Executors.newSingleThreadExecutor();
        es.execute(io);
        System.out.println(Thread.currentThread().getName() + " EXIT");
        es.shutdown();
    }

    static class IntObservable extends Observable implements Runnable {

        @Override
        public void run() {
            for (int i = 1; i <= 10; i++) {
                setChanged();
                notifyObservers(i); // push
            }
        }
    }
}
~~~

위의 코드는 두가지 단점을 가지고 있습니다. 첫번째는 완료라는 개념이 없습니다. 예를 들어서 DB에 데이터를 긁어와서 Push 하는데 긁어온 데이터를 전부 Push 했다면 이를 알릴 방법이 없습니다. 두번째는 Exception을 처리할 수 있는 방법이 없습니다. 과거의 옵저버 패턴은 이런 문제점을 가지고 있기 때문에 현재 Deprecated가 되었고 현재는 두가지 문제점을 보완한 Spring Reactive에서 제공하는 Publusher와 Subscriber를 사용하면 됩니다.

장점은 비동기로 동작하는 코드를 쉽게 작성할 수 있습니다. 반면에 Pull 방식처럼 이터레이터를 사용하면 너무 어려워 집니다. 이런 장점 때문에 Spring Reactive Streams는 개선된 옵저버 패턴을 적극적으로 사용하고 있습니다.


## Push 방식의 Publisher, Subscriber (Spring Reactive)

위의 문제점을 해결하기 위해서 개선된 옵저버 패턴에는 onError, onComplete 메서드가 추가되었고, Subscription이 추가되었습니다. 자세한 스펙은 [이곳](https://github.com/reactive-streams/reactive-streams-jvm/blob/v1.0.3/README.md#specification)에서 확인하실 수 있습니다.

~~~java
public class PubSub {

    public static void main(String[] args) {
        /*
         * Publisher <- Observable
         * Subscriber <- Observer
         *
         * subscription
         * 기존은 onSubscribe() 만 호출하면 됐는데 이제는 subscription 을 넣어서 줘야 합니다.
         * 푸쉬 방식인데 왠 요청?
         * 백 프레셔(역압) publisher 와 subscriber 의 속도차이가 발생하게 되는데 이를 subscription 을 통해서
         * 조절하도록 합니다.
         *
         * 만약, Publisher 는 너무 빠르고 Subscribe 는 하나를 처리하는데 느리다면 어디서 버퍼를 만들거나
         * 정보가 유실될 수 있습니다. 그 반대도 마찬가지이기 때문에 Reqeust 를 통해서 스케일링을 합니다.
         *
         * 이렇게 스케일링은 한다면 여러가지 장점이 있습니다. 메모리 사용량이 피크를 치지 않고 캐쉬에 저장했다가
         * 천천히 끌어가면 항상 일정한 크기를 유지하도록 할 수 있습니다. (넷플릭스에서 적용함)
         * */

        Iterable<Integer> itr = Arrays.asList(1, 2, 3, 4, 5);

        Publisher<Integer> p = new Publisher<Integer>() {
            @Override
            public void subscribe(Subscriber<? super Integer> s) {

                Iterator<Integer> it = itr.iterator();

                s.onSubscribe(new Subscription() {
                    @Override
                    public void request(long n) {
                        while (n-- > 0) {
                            if (it.hasNext()) {
                                s.onNext(it.next());
                            } else {
                                s.onComplete();
                                break;
                            }
                        }
                    }

                    @Override
                    public void cancel() {

                    }
                });
            }
        };

        Subscriber<Integer> s = new Subscriber<Integer>() {

            Subscription subscription;

            @Override
            public void onSubscribe(Subscription s) {
                System.out.println("onSubscribe");
                this.subscription = s;
                this.subscription.request(3);
            }

            @Override
            public void onNext(Integer item) {
                System.out.println("onNext" + item);
                // 3개 받고 조금 쉬었다가 3개 더 받을꺼야라는 코드는 여기서 작성
                System.out.println("breakTime");
                this.subscription.request(1);

            }

            @Override
            public void onError(Throwable t) {
                // Subscriber 는 try-catch 구문이 필요 없음.
                System.out.println("onError");
            }

            @Override
            public void onComplete() {
                System.out.println("onComplete");
            }
        };

        p.subscribe(s);
    }
}
~~~

## 결론

Pull 방식과 Push 방식은 쌍대성을 지닙니다. Spring Reactive Streams에서 개서된 옵저버 패턴을 적극적으로 사용하는 이유는 Push 방식이 비동기적으로 처리하기 수월하고 Pull 방식은 너무 어렵기 때문입니다. 개선된 옵저버 패턴은 완료와 예외를 처리할 수 있도록 방식을 제공해주고 subsription을 통해서 스케일링을 가능하도록 도와줍니다.


## 참고




