---
published: true
title:  "Spring Security - DelegatingFilterProxy와 FilterChainProxy에 대해서 알아보자"
categories:
  - spring
---

## 요약

Request가 들어오면 WAS는 각 request에 맞는 서블릿을 실행합니다. 서블릿 필터 중 하나인 DelegatingFilterProxy는 "springSecurityFilterChain" 이름으로 등록된 빈을 찾고 그 빈에게 보안처리의 책임을 위임한다.

## 서론

미니언 선배님께서 KEEPER R2 프로젝트 인증/인가 부분을 인수인계 해 주신다고 하셔서 멈췄던 Spring Security 공부를 다시 시작했습니다. API 사용은 다 배웠으나 아키텍처가 어떻게 되는지 몰라서 이번 기회에 정리했습니다. 

## Request 처리 과정과 DelegatingFilterProxy

Client가 Request를 보내면 WAS(톰캣, etc..)에게 위임합니다. 그러면 WAS는 각 Request에 맞는 서블릿을 실행합니다.

DelegatingFilterProxy는 서블릿 필터이고, Spring IoC 컨테이너와의 다리 역할을 맡고 있습니다. 표준 서블릿을 구현하고 있으며, 실제로 어떠한 보안처리는 하지 않습니다. 이름에서 알 수 있듯이 오직 "위임"의 책임만을 가지고 있습니다.

![DelegatingFilterProxy](https://github.com/02ggang9/02ggang9.github.io/blob/master/_posts/images/spring/security/chapter2/DelegatingFilterProxy.png?raw=true)

그림에서 볼 수 있듯이 FilterChainProxy에게 Request를 위임하는 모습을 확인하실 수 있습니다.

## FilterChainProxy

FilterChainProxy는 SecurityFilterAutoConfiguration(Auto Configuration)에서 "springSecurityFilterChain"라는 이름으로 빈 등록이 됩니다. 즉, DelegatingFilterProxy는 Application Context에서 "springSecurityFilterChain" 이름을 갖는 빈을 찾고 위임을 할 준비를 합니다.

~~~java
@AutoConfiguration(after = SecurityAutoConfiguration.class)
@ConditionalOnWebApplication(type = Type.SERVLET)
@EnableConfigurationProperties(SecurityProperties.class)
@ConditionalOnClass({ AbstractSecurityWebApplicationInitializer.class, SessionCreationPolicy.class })
public class SecurityFilterAutoConfiguration {

	private static final String DEFAULT_FILTER_NAME = AbstractSecurityWebApplicationInitializer.DEFAULT_FILTER_NAME; // <- 아 값을 확인해 보면 "springSecurityFilterChain" 입니다.

	@Bean
	@ConditionalOnBean(name = DEFAULT_FILTER_NAME)
	public DelegatingFilterProxyRegistrationBean securityFilterChainRegistration(
			SecurityProperties securityProperties) {
		DelegatingFilterProxyRegistrationBean registration = new DelegatingFilterProxyRegistrationBean(
				DEFAULT_FILTER_NAME);
		registration.setOrder(securityProperties.getFilter().getOrder());
		registration.setDispatcherTypes(getDispatcherTypes(securityProperties));
		return registration;
	}

	private EnumSet<DispatcherType> getDispatcherTypes(SecurityProperties securityProperties) {
		if (securityProperties.getFilter().getDispatcherTypes() == null) {
			return null;
		}
		return securityProperties.getFilter()
			.getDispatcherTypes()
			.stream()
			.map((type) -> DispatcherType.valueOf(type.name()))
			.collect(Collectors.toCollection(() -> EnumSet.noneOf(DispatcherType.class)));
	}

}
~~~

실제 DelegatingFilterProxy에서 디버깅해 보면 targetName이 "springSecurityFilterChain"인 것을 확인할 수 있습니다.

![DelegatingFilterProxy](https://github.com/02ggang9/02ggang9.github.io/blob/master/_posts/images/spring/security/chapter2/DelegatingFilterProxy2.png?raw=true)
