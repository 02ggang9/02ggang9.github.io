---
published: false
title:  "Spring Webflux - Testing 방법1"
categories:
  - spring
---

## 서론

Kevin의 알기 쉬운 Spring Reactive 강의 중 Testing 챕터를 공부하고 정리했습니다. 코드를 따라치는데, 뭔가 로직을 테스트 한다기 보다 프레임워크를 테스트 하는 느낌을 많이 받았습니다. 아직 Webflux를 사용하는 프로젝트에 참여해 본적이 없고, Youtube 레퍼런스도 없어 조금 답답하네요..!

## StepVerifier

StepVerifier은 리액티브 프로그래밍에서 테스트하기 위한 도구입니다. 테스트 코드가 mockMvc랑 비슷하게 메서드 체이닝을 거는데, 언제 emit 해야하고, 언제 오류가 발생해야 하는지, 스트림이 언제 complete 해야 하는지 등을 예상하면서 코드를 작성하게 됩니다. 자세한 코드는 아래의 예시에서 살펴보도록 하겠습니다.

### expectNext

Publisher가 특정 값을 emit 할 것으로 예상할 때 사용합니다.

~~~java
class StepVerifierGeneralTestExample01 {

    @Test
    fun sayHelloReactorTest() {
        StepVerifier.create(Mono.just("Hello Reactor"))
            .expectNext("Hello Reactor")
            .expectComplete()
            .verify()
    }

		@Test
    fun divideByTwoTest() {
        val source = Flux.just(2, 4, 6, 8, 10)
        StepVerifier.create(GeneralExample.divideByTwo(source))
            .expectNext(1,2,3,4,5)
            .expectComplete()
            .verify()
    }
}
~~~

### expectNextCount

Publisher가 몇 개의 값을 emit 할지 예상할 때 사용합니다.

~~~java
		@Test
    fun rangeNumberTest() {
        val source = Flux.range(0, 1000)
        StepVerifier.create(source.take(500),
            StepVerifierOptions.create().scenarioName("Verify from 0 to 499"))
            .expectSubscription()
            .expectNext(0)
            .expectNextCount(498)
            .expectNext(499)
            .expectComplete()
            .verify()
    }
~~~

### expectComplete

Publisher가 성공적으로 방출을 완료할 것이라고 예상할 때 사용합니다.

~~~java
		@Test
    fun rangeNumberTest() {
        val source = Flux.range(0, 1000)
        StepVerifier.create(source.take(500),
            StepVerifierOptions.create().scenarioName("Verify from 0 to 499"))
            .expectSubscription()
            .expectNext(0)
            .expectNextCount(498)
            .expectNext(499)
            .expectComplete()
            .verify()
    }
~~~

### as

StepVerifier 테스트에서 추가적인 설명이 필요할 때 사용합니다. 코틀린에서 as는 예약어라 `을 붙여 사용해야 합니다.

~~~java
		@Test
    fun sayHelloReactorTest() {
        StepVerifier.create(GeneralExample.sayHelloReactor())
            .expectSubscription()
            .`as`("# expect subscription")
            .expectNext("Hi")
            .`as`("# expect Hi")
            .expectNext("Reactor")
            .`as`("# expect Reactor")
            .verifyComplete()
    }
~~~

### expectError

Publisher가 특정 타입의 에러를 방출할 것이라 예상할 때 사용합니다. 구체적인 에러 타입을 지정하는 것 또한 가능합니다. 

~~~java
		@Test
    fun occurErrorTest() {
        val source = Flux.just(2, 4, 6, 8, 10)
        StepVerifier.create(GeneralExample.occurError(source))
            .expectNext(1)
            .expectNext(2)
            .expectNext(3)
            .expectNext(4)
            .expectError(ArithmeticException::class.java)
            .verify()
    }
~~~

### advanceTimeBy

가상의 시간을 전진시키고 싶을 때 사용합니다. emit 할 때 시간을 둔다면 이 메서드가 유용하게 쓰입니다.

~~~java
		// 특정 시간만큼 시간을 앞당긴다.
    @Test
    fun getCOVID19CountTest() {
        StepVerifier.withVirtualTime { TimeBasedExample.getCOVID19Count(
            Flux.interval(Duration.ofHours(12)).take(1)
        ) }
            .expectSubscription()
            .then { VirtualTimeScheduler.get().advanceTimeBy(Duration.ofHours(12)) }
            .expectNextCount(11)
            .expectComplete()
            .verify()
    }
~~~

### thenAwait

가상의 시간을 전진시키는 것이 아니라, Publisher가 값을 emit할 때 까지 기다립니다.

~~~java
		@Test
    fun getCOVID19CountTest() {
        StepVerifier.withVirtualTime {
            TimeBasedExample.getCOVID19Count(
                Flux.interval(Duration.ofHours(12)).take(1)
            )
        }
            .expectSubscription()
            .thenAwait(Duration.ofHours(12))
            .expectNextCount(11)
            .expectComplete()
            .verify()
    }
~~~

## 결론

Spring Reactive에서 자주 사용하는 StepVerifier에 대해서 알아봤습니다. 다양한 메서드가 있는데, 메서드 체이닝 형식으로 테스트 코드가 작성됩니다. 아직 서비스 로직을 테스트한 것이 아니라서, 프레임워크 동작을 테스트하는 것 같은 느낌이 드네요! 조만간 졸업과제에서 사용할 때 조금 더 자세하게 글을 작성하도록 하겠습니다. 이번 글은 영양가가 없네요...

