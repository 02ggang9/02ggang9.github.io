---
published: true
title:  "Spring Security - AuthenticationManager, AuthenticationProvider"
categories:
  - spring
---

## AuthenticationManager

자식에서 적절한 AuthenticationProvider를 찾지 못하면 부모 AuthenticationManager로 올라와서 계속해서 탐색합니다.
AuthenticationManager는 AuthenticationProvider 목록 중에서 인증 처리 요건에 맞는 AuthenticationProvider를 찾아 인증처리를 위임하는 책임을 가지고 있습니다. 내부적으로 2개의 ProviderManager를 생성하는데 부모 ProviderManager를 만들어 AuthenticationProvider 탐색을 이어나갈 수 있습니다. KEEPER R2 프로젝트에서는 Jwt 인증 방식을 사용하기 때문에 내부적으로 커스터마이징 해 사용하고 있습니다.


![AuthenticationManager1](https://github.com/02ggang9/02ggang9.github.io/blob/master/_posts/images/spring/security/chapter2/AuthenticationManager1.png?raw=true)

~~~java
public class AuthenticationManagerBuilder
		extends AbstractConfiguredSecurityBuilder<AuthenticationManager, AuthenticationManagerBuilder>
		implements ProviderManagerBuilder<AuthenticationManagerBuilder> {

            @Override
            protected ProviderManager performBuild() throws Exception {
                if (!isConfigured()) {
                    this.logger.debug("No authenticationProviders and no parentAuthenticationManager defined. Returning null.");
                    return null;
                }
                ProviderManager providerManager = new ProviderManager(this.authenticationProviders, // 또 다른 ProviderManager를 생성
                        this.parentAuthenticationManager);
                if (this.eraseCredentials != null) {
                    providerManager.setEraseCredentialsAfterAuthentication(this.eraseCredentials);
                }
                if (this.eventPublisher != null) {
                    providerManager.setAuthenticationEventPublisher(this.eventPublisher);
                }
                providerManager = postProcess(providerManager);
                return providerManager;
            }
        }
~~~


## AuthenticationProvider

AuthenticationProvider는 인터페이스이므로 이를 구현해서 전달받은 Authentication에 대해서 적절한 처리를 할 수 있습니다.

![AuthenticationManager2](https://github.com/02ggang9/02ggang9.github.io/blob/master/_posts/images/spring/security/chapter2/AuthenticationManager2.png?raw=true)

~~~java
public interface AuthenticationProvider {

    // Authentication에 대해서 검증을 한 후 유저의 정보를 저장한 Authentication을 반환합니다.
	Authentication authenticate(Authentication authentication) throws AuthenticationException;

    // Authentication이 처리 가능한지 알려주는 메서드입니다.
	boolean supports(Class<?> authentication);
}
~~~

