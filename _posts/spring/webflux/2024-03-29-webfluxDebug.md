---
published: true
title:  "Spring Webflux - Debugging 방법"
categories:
  - spring
---

## 서론

많은 사람들이 Spring Webflux는 디버깅이 어렵다고 말합니다. 아래의 본론에서 Spring Webflux가 왜 디버깅이 어려운지, 어떻게 디버깅을 할 수 있는지에 대해서 알아보도록 하겠습니다.

## Spring Webflux는 왜 디버깅이 어려운가?

Spring Webflux의 디버깅이 어려운 이유는 하나의 request를 처리할 때 많은 스레드를 거쳐가기 때문입니다. Spring MVC는 request가 들어온다면, 하나의 thread가 response가 나갈 때 까지 전담하게 됩니다. 자바는 스택 트레이스로 에러의 원인을 파악하는데 각 스레드는 자신만의 스택 영역을 가지고 있어 thread A에서 thread B로 넘어갈 때 문제가 발생하는 것 입니다.

## Webflux Debugging 방법

### Hooks.onOperatorDebug()

Hooks.onOperatorDebug는 웹 어플리케이션 전체에 디버깅 모드를 적용하게 됩니다. 이 방식은 리소스가 많이 들기 때문에 prod 환경에서는 절대로 사용할 수 없을 것 같습니다.

~~~java
fun main() {
    Hooks.onOperatorDebug()
    val fruits = mapOf(
        "banana" to "바나나",
        "apple" to "사과",
        "pear" to "배",
        "grape" to "포도"
    )

    Flux.fromArray(arrayOf("BANANAS", "APPLES", "PEARS", "MELONS"))
        .map { it.lowercase() }
        .map { it.substring(0, it.length-1) }
        .map { fruits[it] }
        .map { "맛있는 $it" }
        .subscribe { println("onNext: $it") }
}
~~~

![Hooks](https://github.com/02ggang9/02ggang9.github.io/blob/master/_posts/images/spring/webflux/debug/hooks.png?raw=true)


### checkpoint()

checkpoint 방식은 전역적으로 디버깅 모드를 적용하는 것이 아니라 관심있는 Publisher만 디버깅 모드를 적용할 수 있습니다. 또, 파라미터로 이름을 설정할 수 있습니다.

~~~java
fun main() {
    Flux.just(2, 4, 6, 8)
        .zipWith(Flux.just(1, 2, 3, 0)) { a, b -> a / b }
        .checkpoint("CheckpointExample02.zipWith.checkPoint", true)
        .map { it + 2 }
        .checkpoint("CheckpointExample02.map.checkPoint")
        .subscribe { println("onNext: $it")}
}
~~~

![Hooks](https://github.com/02ggang9/02ggang9.github.io/blob/master/_posts/images/spring/webflux/debug/checkpoint.png?raw=true)


### log()

log 방식은 checkpoint와 마찬가지로 관심있는 publisher에 디버깅 모드를 적용할 수 있습니다. 추가된 기능은 publisher를 구독하고 일어나는 event를 상세히 출력하는 것 입니다.

~~~java
fun main() {
    val fruits = mapOf(
        "banana" to "바나나",
        "apple" to "사과",
        "pear" to "배",
        "grape" to "포도"
    )

    Flux.fromArray(arrayOf("BANANAS", "APPLES", "PEARS", "MELONS"))
        .log("Fruit.Source")
        .map { it.lowercase() }
        .log("Fruit.Lower")
        .map { it.substring(0, it.length-1) }
        .log("Fruit.Substring")
        .map { fruits[it] }
        .log("Fruit.Name")
        .subscribe { println("onNext: $it") }
}
~~~

![Hooks](https://github.com/02ggang9/02ggang9.github.io/blob/master/_posts/images/spring/webflux/debug/log.png?raw=true)


## 결론

Spring Webflux에서 디버깅이 어려운 이유는 하나의 request가 여러 스레드에 의해 처리되기 때문입니다. Spring Webflux는 이런 문제를 해결하기 위해 3가지 방법을 제시하고 있는데 Hooks 연산자는 어플리케이션 전역에 디버깅 모드를 적용하기 때문에 많은 부하를 줄 수 있습니다. checkpoint 연산자는 관심있는 publisher만 디버깅 모드를 적용할 수 있고, log 연산자는 좀 더 상세한 event를 출력하는 기능을 제공합니다.
