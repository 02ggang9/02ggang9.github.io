---
published: true
title:  "JPA - OSIV 기능에 대해서 알아보자."
categories:
  - Jpa
---

KEEPER R2 프로젝트에서 이메일 변경 API가 정상적으로 작동하지 않는다는 제보를 받았습니다. 근본적인 이유는 @Transactional 어노테이션을 빼먹어서 생긴 문제이지만 엉뚱한 삽질로 OSIV에 대해서 공부하게 되었고 JPA에 대해서 더욱 잘 알게 되었습니다. 아래는 트러블슈팅 과정에서 공부한 OSIV에 대해서 알아보고 어떤 엉뚱한 생각을 했는지 알아보겠습니다.

## 스프링 컨테이너의 기본 전략
스프링 컨테이너는 트랜잭션 범위의 영속성 컨텍스트 전략을 사용합니다. 말 그대로 트랜잭션의 범위와 영속성 컨텍스트가 유지되는 생존 범위가 같다는 의미입니다. 트랜잭션이 시작되면 영속성 컨텍스트가 생성되고 트랜잭션이 끝날 때 영속성 컨텍스트는 종료되게 됩니다.

스프링 컨테이너는 스레드 마다 각각 다른 트랜잭션을 할당하기 때문에 멀티 스레드 상황에서 안전합니다.

### 준영속 상태와 지연로딩
프리젠테이션 계층에서 준영속 상태를 지연로딩을 하게 되면 LazyInitializationException 예외가 발생합니다. 이유는 대부분 서비스 계층에서 트랜잭션이 시작하고 종료되기 때문에 프리젠테이션 계층에서 영속성 컨텍스트는 종료된 상태입니다.

![OSIVOff](https://github.com/02ggang9/02ggang9.github.io/blob/master/_posts/images/OSIV/OsivOff.png?raw=true)<sup>[1](#1)</sup>

위의 지연로딩 문제점을 OSIV를 사용해 해결할 수 있습니다.

## 과거 OSIV(Open Session in View)
과거 OSIV는 트랜잭션의 범위가 Filter Interceptor 까지 였기때문에 영속성 컨텍스트의 범위도 프리젠테이션 계층까지 넓어졌습니다. 따라서 영속성 컨텍스트가 살아있기 때문에 프리젠테이션 계층에서도 지연로딩을 사용할 수 있습니다.

![PastOSIV](https://github.com/02ggang9/02ggang9.github.io/blob/master/_posts/images/OSIV/PastOSIV.png?raw=true)<sup>[2](#2)</sup>

하지만 의도치 않게 값이 변경되는 장애가 발생할 수 있습니다. 트랜잭션의 범위도 프리젠테이션 계층까지였기 때문에 비지니스 로직을 거치고 프리젠테이션 계층에서 엔티티의 값을 변경하면 변경 감지로 인해서 값이 수정될 수 있습니다.

위의 문제점을 해결하기 위해서는 전통적으로 사용되는 DTO를 사용해 값만 반환하는 방법이 있습니다. 하지만 이런 DTO 사용도 엔티티를 거의 복사한 듯한 클래스를 만들어야 하기 때문에 코드의 양이 증가되는 단점도 있습니다.

## Spring OSIV(Open Session in View)
### 트랜잭션 없이 읽기
현재 사용되는 Spring OSIV는 비지니스 계층에서만 트랜잭션을 사용합니다. 때문에 비지니스 로직을 거치고 프리젠테이션 계층에서 엔티티의 값을 수정해도 변경 감지로 인한 데이터 변경은 불가능 합니다. 하지만 영속성 컨텍스트는 살아있기 때문에 지연 로딩을 사용해 엔티티를 조회할 수 있는 장점이 생깁니다. 이를 "트랜잭션 없이 읽기(NonTransactional reads)"라고 합니다.

![SpringOSIV](https://github.com/02ggang9/02ggang9.github.io/blob/master/_posts/images/OSIV/SpringOSIV.png?raw=true)<sup>[3](#3)</sup>

## 문제점
### 비즈니스 로직 실행 전 엔티티 값을 변경하면 안된다.
비즈니스 로직 실행 전 엔티티 값을 변경하게 된다면 변경 감지로 인해서 엔티티의 값이 수정됩니다. 클라이언트에서 요청이 들어오면 미리 영속성 컨텍스트를 만들기 때문에 수정된 엔티티가 영속성 컨텍스트에 저장되고 비즈니스 로직이 끝난 후 트랜잭션이 끝날 때 변경 감지로 인해서 데이터 값이 수정됩니다.

### 데이터베이스 커넥션이 부족할 수 있다.
대규모 실시간 API에서 데이터베이스 커넥션이 말라버려서 장애로 이어질 수 있습니다. 트랜잭션이 시작될 때 데이터베이스 커넥션을 맺게 되는데 데이터베이스 커넥션을 반납하는 시점은 클라이언트에게 Response가 전달될 때 입니다. 따라서 프리젠테이션 계층에서 외부 API를 호출했는데 Blocking 걸리는 등 여러가지 이유로 응답이 늦어지면 데이터베이스 커넥션 리소스를 갉아먹고 장애로 이어질 수 있습니다.

이렇게 실시간 트래픽이 중요한 경우에는 OSIV를 끄는 것이 좋습니다.

## 마치며
KEEPER 프로젝트에서는 @LoginMember 어노테이션을 애용하고 있습니다.

~~~java
Authentication authentication = SecurityContextHolder.getContext().getAuthentication();
long loginMemberId = Long.parseLong(authentication.getName());
return memberRepository.findById(loginMemberId)
    .orElseThrow(() -> new BusinessException(loginMemberId, "JWT", ErrorCode.MEMBER_NOT_FOUND));
~~~

@LoginMember 어노테이션은 ThreadLocal에 저장된 SecurityContext에서 Authentication 정보를 가져오고 데이터베이스에서 조회하는 과정을 거치고 있습니다. 이 과정은 프리젠테이션 계층에서부터 시작되기 때문에 OSIV를 몰랐을 때는 영속성 컨텍스트에 member 엔티티가 저장되지 않아서 더티체킹이 안되는 오류라고 생각했습니다. 하지만 KEEPER 프로젝트에서는 OSIV가 기능이 켜져있기 때문에 영속성 컨텍스트에 잘 저장이 되어 있었고 다른 근본적인 원인을 찾아 해결했습니다.

### 참고문헌
자바 ORM 표준 JPA 프로그래밍 - 김영한 지음 - 2015.07.20

<a name="1">[1]</a> (참고: 자바 ORM 표준 JPA - 김영한)

<a name="2">[2]</a> URL: https://hudi.blog/multi-datasource-issue-with-osiv/

<a name="3">[3]</a> (참고: 자바 ORM 표준 JPA - 김영한)




