---
published: false
title:  "Spring Security - Authentication과 SecurityContextHolder, SecurityContext"
categories:
  - spring
---

## Authentication

Authentication은 사용자의 인증 정보입니다. Authentication의 역할은 인증 시 사용자의 id값과 password를 담고 인증 절차를 위해 전달하는 역할이 있고, 인증 절차 후 User 객체와 권한 정보를 가지고 SecurityContext에 저장됩니다. 이 저장된 값은 전역적으로 참조할 수 있기 때문에 아주 유용하게 쓰입니다.

아래는 Authentication Flow 입니다.

![DelegatingFilterProxy](https://github.com/02ggang9/02ggang9.github.io/blob/master/_posts/images/spring/security/chapter2/Authentication1.png?raw=true)

~~~java
public class UsernamePasswordAuthenticationFilter extends AbstractAuthenticationProcessingFilter {

	public static final String SPRING_SECURITY_FORM_USERNAME_KEY = "username";

	public static final String SPRING_SECURITY_FORM_PASSWORD_KEY = "password";

	private static final AntPathRequestMatcher DEFAULT_ANT_PATH_REQUEST_MATCHER = new AntPathRequestMatcher("/login",
			"POST");

	@Override
	public Authentication attemptAuthentication(HttpServletRequest request, HttpServletResponse response)
			throws AuthenticationException {
		if (this.postOnly && !request.getMethod().equals("POST")) {
			throw new AuthenticationServiceException("Authentication method not supported: " + request.getMethod());
		}
		String username = obtainUsername(request);
		username = (username != null) ? username.trim() : ""; // Username 추출
		String password = obtainPassword(request);
		password = (password != null) ? password : ""; // Password 추출
		UsernamePasswordAuthenticationToken authRequest = UsernamePasswordAuthenticationToken.unauthenticated(username,
				password);
		// Allow subclasses to set the "details" property
		setDetails(request, authRequest);
		return this.getAuthenticationManager().authenticate(authRequest); // Authentication 객체를 리턴
	}
}
~~~

## SecurityContextHolder




## SecurityContext



