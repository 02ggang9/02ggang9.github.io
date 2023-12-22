---
published: true
title:  "Spring - ControllerAdvice를 사용해 예외를 효과적으로 처리하자."
categories:
  - spring
---

## 요약

스프링에서는 @ControllerAdvice를 통해 효과적으로 예외 처리를 할 수 있습니다. 

## 서론
플랫폼기반프로그래밍(JAVA) 전공 수업에서 예외 처리에 관해서 배웠습니다. 과제를 할 때 try-catch로 ExceptionHadling을 했는데 코드의 길이가 길어지고 중복된 코드가 많아지는 문제점이 있었습니다.

하지만 전에 했었던 KEEPER R2 프로젝트에서는 try-catch를 활용한 Exception Handling 코드를 작성한 기억이 없었습니다. 제가 맡았던 상벌점 도메인에서 상점을 등록할 때 이미 동일한 이름의 상점이 있을 때 예외를 던졌지만, 상위 메서드에서 후속 처리를 하지 않았습니다. 이에 의문점을 가지고 미니언 선배님께서 작성한 코드를 살펴보고 정리해 봤습니다.

아래에는 KEEPER R2 프로젝트의 코드를 발췌해 스프링에서는 어떻게 예외처리를 할 수 있는지 짧게 알아보도록 하겠습니다.

## Controller Advice

@ExceptionHandler는 메서드는 해당 메서드가 선언된 @Controller 클래스에만 적용됩니다. 대신 @ControllerAdvice 또는 @RestControllerAdvice 클래스에 선언된다면 모든 @Controller 클래스에 적용됩니다.[[1]](https://docs.spring.io/spring-framework/reference/web/webmvc/mvc-controller/ann-advice.html) KEEPER R2 프로젝트에서는 @RestControllerAdvice 클래스에서 BusinessException을 Handling 하는 코드를 작성했습니다. 이렇게 한다면 서비스 레이어에서 BusinessException을 throw 해 예외 처리를 회피하면 컨트롤러 레이어에서 예외가 발생하고 @ExceptionHandler에 정의된 메서드가 실행되어 예외를 처리하게 됩니다.

~~~java
// Service Layer

@Transactional
public void deleteMeritLog(long meritLogId) {
    meritLogRepository.findById(meritLogId)
        .orElseThrow(() -> new BusinessException(meritLogId, "meritLogId", MERIT_LOG_NOT_FOUND));

    meritLogRepository.deleteById(meritLogId);
}
~~~

~~~java
@RestControllerAdvice
public class ExceptionAdvice {

    @ExceptionHandler(BusinessException.class)
    public ResponseEntity<ErrorResponse> businessException(BusinessException e) {
        String errorMessage = getErrorMessage(e.getInvalidValue(), e.getFieldName(), e.getMessage());
        return ResponseEntity.status(e.getHttpStatus())
            .body(ErrorResponse.from(errorMessage));
    }
}
~~~

>예외 처리 회피  예외 처리를 자신이 담당하지 않고 자신을 호출한 쪽으로 던져버리는 것이다. throws 문으로 선언해서 예외가 발생하면 알아서 던져지게 하거나 catch 문으로 일단 예외를 잡은 후에 로그를 남기고 다시 예외를 던지는 것이다. 예외 처리를 회피하려면 반드시 다른 오브젝트나 메서드가 예외를 대신 처리할 수 있도록 해야 한다.  토비의 스프링 3.1 p.286 

## 결론

try-catch를 매번 사용한다면 코드의 길이가 길어지고 중복 코드가 발생합니다. 이를 해결하기 위해서 스프링에서 제공하는 @ControllerAdvice를 사용했습니다. 코드의 중복이 없어지고 유지보수가 편하게 바뀌었습니다.

얼렁뚱땅 넘어간 점이 있는데 @RestControllerAdvice는 단지 @ControllerAdvice에 @ResponseBody를 붙인 것 뿐입니다. KEEPER R2 프로젝트처럼 REST API로 개발하신다면 @RestControllerAdvice를 사용하시면 되고 view를 랜더링 해야 한다면 @ControllerAdvice를 사용하시면 됩니다.
