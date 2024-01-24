---
published: false
title:  "Spring Boot - 자동 구성 기반 애플리케이션"
categories:
  - spring
---


## 메타 애노테이션과 합성 애노테이션

### 메타 애노테이션
메타 애노테이션은 애노테이션에 붙이는 애노테이션입니다. 대표적인 예시로 @Controller가 있습니다. @Controller라는 애노테이션에 @Componet 애노테이션이 붙었는데 기능적으로 @Component와 차이점이 없습니다. 하지만 부가적인 효과를 기대할 수 있는데 @Controller를 예로 들면 @Controller는 Controller의 역할을 하는 빈이라는 것을 알 수 있습니다. @Service도 마찬가지 입니다.

~~~java
@Target(ElementType.TYPE)
@Retention(RetentionPolicy.RUNTIME)
@Documented
@Component
public @interface Controller {

	@AliasFor(annotation = Component.class)
	String value() default "";

}
~~~

### 합성 애노테이션
합성 애노테이션은 두 개 이상의 메타 애노테이션을 합친 애노테이션입니다. 아주 친숙한 @RestController도 합성 애노테이션입니다.

~~~java
@Target(ElementType.TYPE)
@Retention(RetentionPolicy.RUNTIME)
@Documented
@Controller // Meta Annotation
@ResponseBody // Meta Annotation
public @interface RestController {

	@AliasFor(annotation = Controller.class)
	String value() default "";
}
~~~

## 빈 오브젝트의 역할과 구분

### 컨테이너 인프라 스트럭처 빈
컨테이너 인프라 스트럭처 빈은 스프링 컨테이너 자신이거나 기능이 확장되면서 추가된 빈입니다. 

### 애플리케이션 로직 빈
애플리케이션 로직 빈은 HelloController와 HelloSimpleService 처럼 애플리케이션의 기능이나 비지니스 로직을 다고 있는 빈들입니다.

### 애플리케이션 인프라 빈
대부분 우리가 직접 작성하지는 않지만, 아래처럼 명시적으로 빈 구성정보를 작성해야 정상작동 됩니다.

~~~java
@Bean
public ServletWebServerFactory servletWebServerFactory() {
  return new TomcatServletWebServerFactory();
}
~~~


