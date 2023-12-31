---
published: true
title:  "Spring Security - 스프링 시큐리티 기본 API 및 Filter 이해"
categories:
  - spring
---

## Form Login 인증

Spring Boot의 버전이 업그레이드 됨에 따라 config를 작성하는 방법이 달라졌습니다. 과거에는 WebSecurityConfigurerAdapter를 상속받았지만 최근에는 SecurityFilterChain을 빈으로 등록해 사용하는 것을 권장하고 있습니다. 이와 관련된 설명은 [스프링 공식 문서](https://spring.io/blog/2022/02/21/spring-security-without-the-websecurityconfigureradapter/)에서 확인하실 수 있습니다.

![폼 인증 아키텍처](https://github.com/02ggang9/02ggang9.github.io/blob/master/_posts/images/spring/security/chapter1/formLogin구조.png?raw=true)

~~~java
@Configuration
@EnableWebSecurity
public class SecurityConfig {

    @Bean
    public SecurityFilterChain filterChain(HttpSecurity http) throws Exception {
        return http.authorizeHttpRequests(auth -> auth.
                        anyRequest().authenticated()
                )
                .formLogin(formLogin -> formLogin
                        .loginPage("/loginPage") // 이 페이지는 로그인을 위한 페이지이므로 누구나 접근이 가능해야 한다.
                        .defaultSuccessUrl("/") // 로그인에 성공하면 redirect
                        .failureUrl("/login") 
                        .usernameParameter("userId") // 주의
                        .passwordParameter("password") // 주의
                        .loginProcessingUrl("/login_proc")
                        .successHandler((request, response, authentication) -> {
                            System.out.println("authentication" + authentication.getName());
                            response.sendRedirect("/");
                        })
                        .failureHandler((request, response, exception) -> {
                            System.out.println("exception" + exception.getMessage());
                            response.sendRedirect("/login");
                        })
                        .permitAll()
                )
                .build();
    }

}
~~~

## 권한 설정과 표현식

이 부분도 과거와 달라졌습니다. 현재는 requestMatcher를 사용하는 것을 권장하고 있습니다. 그리고 초기 프로젝트에서 권한 관련한 부분을 테스트 하기 위해서 인메모리 방식으로 유저를 등록하는 방법이 있습니다.

~~~java
@Configuration
@EnableWebSecurity
public class SecurityConfig {

    @Bean
    public UserDetailsManager users() { // 인메모리 방식으로 User 등록
        UserDetails user = User.builder()
                .username("user")
                .password("{noop}1111")
                .roles("USER")
                .build();

        UserDetails sys = User.builder()
                .username("sys")
                .password("{noop}1111")
                .roles("SYS")
                .build();

        UserDetails admin = User.builder()
                .username("admin")
                .password("{noop}1111")
                .roles("ADMIN", "SYS", "USER")
                .build();

        return new InMemoryUserDetailsManager(user, sys, admin);
    }

    @Bean
    public SecurityFilterChain filterChain(HttpSecurity http) throws Exception {
        return http.authorizeHttpRequests(auth -> auth // 권환 체크
                        .requestMatchers("/user").hasRole("USER")
                        .requestMatchers("/admin/pay").hasRole("ADMIN")
                        .requestMatchers("/admin/**").hasAnyRole("ADMIN", "SYS")
                        .anyRequest().authenticated()
                )
    }
}
~~~

## 예외 처리 및 요청 캐시 필터

예외를 처리하는 아키텍처는 아래의 사진과 같습니다.

![예외처리 아키텍처](https://github.com/02ggang9/02ggang9.github.io/blob/master/_posts/images/spring/security/chapter1/예외처리.png?raw=true)

~~~java
@Configuration
@EnableWebSecurity
public class SecurityConfig {

    @Bean
    public SecurityFilterChain filterChain(HttpSecurity http) throws Exception {
        return http.authorizeHttpRequests(auth -> auth
                        .requestMatchers("/login").permitAll()
                        .requestMatchers("/user").hasRole("USER")
                        .requestMatchers("/admin/pay").hasRole("ADMIN")
                        .requestMatchers("/admin/**").hasAnyRole("ADMIN", "SYS")
                        .anyRequest().authenticated()
                )
                .formLogin(formLogin -> formLogin // 사용자의 요청관련 정보 저장
                        .successHandler((request, response, authentication) -> {
                            HttpSessionRequestCache requestCache = new HttpSessionRequestCache();
                            SavedRequest savedRequest = requestCache.getRequest(request, response);
                            String redirectUrl = savedRequest.getRedirectUrl();
                            response.sendRedirect(redirectUrl);
                        }))
                .exceptionHandling(exception -> exception
//                        .authenticationEntryPoint((request, response, authException) -> { // 인증실패 시 처리
//                            response.sendRedirect("/login");
//                        })
                        .accessDeniedHandler(((request, response, accessDeniedException) -> { // 인증실패 시 처리
                           response.sendRedirect("/denied");
                        }))
                )
                .build();
    }
}
~~~
