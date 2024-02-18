---
published: true
title: "Kotlin - Reflection And Dependeny Injection"
categories:
  - Java
---

## 서론

이번에 BDD 소모임에서 코틀린 스터디에 참여했습니다. 보통 언어나 프레임워크를 학습할 때 인터넷 강의를 많이 참고하는데 강의 내용 중 "리플랙션 활용 - 나만의 DI 컨테이너 만들기"라는 흥미로운 챕터를 봐서 바로 결제했습니다. 평소 DI 컨테이너를 구현해 보고 싶었고, 리플랙션에 대해서 공부해 본적이 없어 55,000원 이라는 가격이 아깝지 않았습니다. 아래는 리플랙션 개념, Spring Framework에서 리플랙션을 사용하는 곳과 이유에 대해서 알아보고 마지막으로 우테코 프리코스인 로또 미션에 DI를 적용해 보는 과정에 대해서 알아보고 글을 마치도록 하겠습니다.


## 리플랙션이란

> 간단히 말해 리플랙션이란 런타임에 동적으로 프로퍼티와 메서드에 접근할 수 있는 방법이다. 보통 객체의 메서드나 프로퍼티에 접근할 때는 소스코드 안에 구체적인 선언이 있는 메서드나 프로퍼티 이름을 사용하며, 컴파일러는 그런 이름이 실제로 가르키는 선언을 컴파일 시점에 정적으로 찾아내서 해당하는 선언이 실제 존재함을 보장한다. (Kotlin in Action p.446)

지난 프로젝트를 돌아봤을 때 리플렉션 기술을 사용한 적은 없었고, 본적도 없습니다. 대부분의 개발자 분들은 프로젝트 코드에서 리플랙션을 사용하지 않으셨을 겁니다. 하지만 Spring Framework 같은 곳에서는 리플랙션 기술을 정말 많이 사용한다고 합니다.

## Spring Framework에서 리플랙션을 사용하는 곳과 이유

Spring Framework에서 빈을 등록할 때 리플랙션 기술을 사용합니다. 리플랙션을 사용함으로써 런타임에 클래스 이름 기반으로 빈을 인스턴스화 할 수 있고 객체 간의 강결합을 막아주기 때문에 사용한다고 합니다. 

아래의 Spring Framework 코드를 보시면 인스턴스를 생성할 때 거의 모든 곳에서 리플랙션을 사용하는 것을 확인하실 수 있습니다. 생성자의 파라미터 개수가 몇 개인지, 파라미터 타입이 무엇인지 등의 정보를 런타임에 알 수 있도록 해 줍니다.

![Relfection 이미지](https://github.com/02ggang9/02ggang9.github.io/blob/master/_posts/images/java/kt/reflection/reflection1.png?raw=true)

## Dependency Injection 구현

### packageName 기준으로 Component 스캔 후 컨테이너에 저장

~~~java

fun start(clazz: KClass<*>) {
    val reflections = Reflections(clazz.packageName)
    val jClasses = reflections.getTypesAnnotatedWith(Component::class.java)
    jClasses.forEach { jClasses -> ContainerV2.register(jClasses.kotlin) }
}

private val KClass<*>.packageName: String
    get() {
        val qualifiedName = this.qualifiedName ?: throw IllegalArgumentException("익명 객체입니다!")
        val hierarchy = qualifiedName.split(".")
        return hierarchy.subList(0, hierarchy.lastIndex).joinToString(".")
    }
~~~

packageName에서 추출한 패키지 명을 기준으로 리플랙션을 수행하도록 합니다. 또, @Component 어노테이션이 붙은 모든 클래스를 찾아내고 컨테이너에 등록하도록 했습니다.


### 컨테이너에서 인스턴스 가져오기

~~~java

object ContainerV2 {
    // 등록한 클래스를 보관! = KClass를 보관

    private val registeredClasses = mutableSetOf<KClass<*>>()
    private val cachedInstances = mutableMapOf<KClass<*>, Any>()

    fun register(clazz: KClass<*>) { // Set 자료구조에 저장
        registeredClasses.add(clazz)
    }

    fun <T : Any> getInstance(type: KClass<T>) : T {
        if (type in cachedInstances) { // 싱글톤임을 보장하기 위해서 cache를 사용
            return type.cast(cachedInstances[type])
        }

        // Cached 된 KClass가 없다면 캐싱 후 인스턴스를 반환하도록 함
        val instance =  registeredClasses.firstOrNull { clazz -> clazz == type } // 컨테이너에 가져오고 싶은 클래스 타입이 있는지 확인
                ?.let { clazz -> instantiate(clazz) as T } // 클래스 타입이 있다면 리플랙션을 사용해 인스턴스를 반환하도록 함
                ?: throw IllegalArgumentException("해당 인스턴스 타입을 찾을 수 없습니다.") // 컨테이너에 가져오고 싶은 클래스 타입이 없는 경우 예외를 던지도록 함

        cachedInstances[type] = instance
        return instance
    }

    private fun <T : Any> instantiate(clazz: KClass<T>): T {
        val constructor = findUsableConstructor(clazz) // 사용할 수 있는 생성자가 있는지 확인 함
        val params = constructor.parameters
                .map { param -> getInstance(param.type.classifier as KClass<*>) } // 인스턴스를 생성하기 위해 필요한 모든 클래스를 가져옴
                .toTypedArray()

        return constructor.call(*params) // 생성자를 이용해 인스턴스를 생성
    }

    // clazz의 constructor 들 중, 사용할 수 있는 constructor
    // constructor 에 넣어야 하는 타입들이 모두 등록된 경우(컨테이너에서 관리하고 있는 경우를 의미)
    private fun <T : Any> findUsableConstructor(clazz: KClass<T>): KFunction<T> =
            clazz.constructors.firstOrNull { constructor -> constructor.parameters.isAllRegistered } // Constructor의 파라미터의 타입이 컨테이너에 등록되었는지 확인함.
                    ?: throw IllegalArgumentException("사용할 수 있는 생성자가 없습니다") // 만약 생성자의 파라미터 타입들이 컨테이너에서 관리되고 있지 않으면 예외를 발생하도록 함

    private val List<KParameter>.isAllRegistered: Boolean
        get() = this.all { it.type.classifier in registeredClasses }
}

~~~

### 컨테이너에서 가져온 인스턴스를 실행

인스턴스를 생성할 때 필요한 다른 객체들을 컨테이너에서 자동으로 주입하고 완성된 인스턴스를 반환합니다.

~~~java

class Application

fun main() {
    start(Application::class)
    val instance = ContainerV2.getInstance(BService::class)
    instance.print()
}
~~~

~~~java
@Component
class AService {
    fun print() {
        println("A Service 입니다")
    }
}

@Component
class BService (
        private val aService: AService,
        private val cService: CService?,
) {

    constructor(aService: AService): this(aService, null)

    fun print() {
        this.aService.print()
    }
}

class CService
~~~

## 결론

리플랙션은 런타임에 동적으로 클래스의 메서드나 프로퍼티에 접근하고 싶을 때 사용하는 기술입니다. 대표적으로 Spring Framework에서 Bean을 등록할 때 사용하는데 이는 클래스 간의 강결합을 막아주고, 클래스 이름을 기반으로 빈을 등록 해 주기 위함입니다. 마지막으로 코틀린 언어로 정말 간단한 DI 컨테이너를 만들어 봤습니다.

앞으로 살아가면서 리플랙션을 사용하는 일은 거의 없을 것입니다. 하지만 런타임에 동적으로 어떤 행동을 해야 할 일이 있을 때 리플랙션이라는 기술을 떠올릴 수 있다면 이번 학습이 성공적이었다고 생각합니다.

