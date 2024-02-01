---
published: true
title:  "KEEPER - 로그아웃 불가 및 게시글 확인 불가 문제 해결"
categories:
  - keeper
---


## 서론

KEEPER R2 프로젝트가 배포되고 약 3개월 동안 유지보수가 되지 않고 있었습니다. 이유는 프로젝트 PM님의 바쁜 학기 일정과 선배님의 회사 일이 바빠졌기 때문입니다. 방학이 시작되고 BDD 소모임에서 KEEPER 홈페이지 유지보수를 담당하게 되어 3일 전부터 본격적으로 유지보수를 시작하게 되었습니다. 이틀 전에 KEEPER R2 Release v1.0.1 버전을 Prod 서버에 배포하고 잠에 들었는데 로그아웃이 안되는 문제와 게시글을 확인할 수 없다는 제보를 받고 긴급 핫픽스에 들어갔습니다.

아래의 글에서는 버그가 발생한 복합적인 이유에 대해서 알아보고 삽질, 해결 방법과 앞으로 유지보수 해야 할 일에 대해서 알아보고 글을 마치도록 하겠습니다.


## KEEPER ReadME Quick Start 제작 및 세미나 코드 수정

KEEPER ReadME Quick Start는 하나의 쉘 커맨드만 실행시켜도 KEEPER R2 홈페이지가 로컬에서 동작할 수 있도록 만드는 도구입니다. 만들게 된 계기는 지난 학기 때 한상곤 교수님의 공학작문 및 발표 수업에서 ReadME 작성법에 대해서 배웠는데 KEEPER R2 Backend Repository에는 ReadME가 작성되어 있지 않아 이번 겨울 방학 때 만들어야겠다고 결심했습니다.

KEEPER ReadME Quick Start Tool은 Doo-re Infra를 담당하면서 학습한 쉘 스크립트와 도커 컨테이너 기술을 이용하는데 Spring Boot 컨테이너를 띄울때 이상하게도 세미나 관련한 테스트가 깨지는 현상을 발견했습니다. 해당 현상을 ReadME를 제작하고 있던 PM님께 말씀을 드렸고 세미나 관련한 코드에 문제가 있다는 점을 확인했습니다. 관련된 PR은 [Feature/#397 세미나 테스트코드 오류 수정](https://github.com/KEEPER31337/Homepage-Back-R2/pull/400)에서 확인하실 수 있습니다.

마침 1월, 2월이고 곧 있으면 세미나 관련 행사가 열리기 때문에 문제를 인지하자마자 픽스 후 운영서버에 릴리즈 했습니다. 그 후 잠이 들었고 다음날 로그아웃과 게시글 확인 기능이 안된다는 [제보](https://github.com/KEEPER31337/Homepage-Front-R2/issues/878)를 PM님께 받았습니다.


## 로그아웃과 게시글 확인이 안되는 이유

어제 버그 픽스 후 릴리즈한 코드는 세미나 쪽만 수정했기 때문에 잘 동작하고 있던 로그아웃과 게시글 확인 기능에는 아무런 문제가 없을텐데라고 생각하면서 확인해 보니 정말로 로그인 후 로그아웃과 게시글 확인이 불가능했습니다. 개발자 도구를 열어 에러 메시지를 확인해보니 로그아웃과 게시글을 확인할 때 401이 떨어지고 있었고, 쿠키 저장소를 살펴보니 accessToken과 refreshToken이 비어있었습니다. 

![image1](https://github.com/02ggang9/02ggang9.github.io/blob/master/_posts/images/keeper/cookieExpried/image1.png?raw=true)


로그아웃을 포함한 대부분의 Request는 Spring Security Filter에 걸리게 되는데 accessToken이 없어 401이 떨어졌고 그로 인해서 로그아웃을 못 하는 현상이 일어났습니다.

~~~java
@Bean
public SecurityFilterChain filterChain(HttpSecurity http) throws Exception {
    http.authorizeHttpRequests()
        .requestMatchers("/docs/**", "/keeper_files/**", "/auth-test", "/sign-up/**", "/error",
            "/about/**", "/sign-in/**", "/posts/recent", "/posts/trend")
        .permitAll()
        .anyRequest().hasRole("회원") // accessToken이 없어 401이 떨어짐
        .and()
        .httpBasic().disable()
        .csrf().disable()
        .logout().disable()
        .cors().configurationSource(corsConfigurationSource())
        .and()
        .sessionManagement().sessionCreationPolicy(SessionCreationPolicy.STATELESS)
        .and()
        .exceptionHandling().accessDeniedHandler(customAccessDeniedHandler)
        .and()
        .exceptionHandling().authenticationEntryPoint(customAuthenticationEntryPoint)
        .and()
        .addFilterBefore(jwtAuthenticationFilter, UsernamePasswordAuthenticationFilter.class)
        .addFilterBefore(refreshTokenFilter, JwtAuthenticationFilter.class);

    return http.build();
}
~~~


## Dev 서버는 정상으로 동작(삽질)

![image1](https://github.com/02ggang9/02ggang9.github.io/blob/master/_posts/images/keeper/cookieExpried/image2.png?raw=true)


Dev 서버에는 정상적으로 accessToken과 refreshToken이 발급되고 쿠키에 저장이 되지만 Prod 서버에는 토큰 발급은 되지만 저장이 안되는 것을 확인했습니다. Dev 서버와 Prod 서버의 코드는 동일한 코드(나중에 다르다는 사실을 알게 됨)임에도 불구하고 결과 값이 다르니 프론트와 인프라의 코드 변경으로 인해 발생한 문제인줄 알았습니다. 최근 PR을 싹다 뒤져보고 Chrome의 쿠키 정책이 바뀌웠는지도 찾아봤지만 아무런 소득도 얻지 못했습니다.



## CICD는 실패해도 코드가 머지되는 문제

KEEPER는 CICD 코드가 PR이 Merge될 때 자동으로 트리거 됩니다. 이 말은 CICD가 실패해 Dev 서버에 배포가 되지 않아도 코드는 머지된다는 소리입니다. 2달 전에 미니언 선배님께서 올린 PR을 2주 전에 제가 자기 전에 아무 생각 없이 Merge 시켰고 CICD 결과가 실패했다는 결과를 까먹고 있었습니다. 

핫픽스할 때 세미나 관련한 코드만 Prod 서버에 반영된 줄 알았지만 사실 다른 코드도 포함되어 반영된 것 입니다. 이 사실을 깨닫고 2개월 전 PR에서 수정된 코드를 분석하기 시작했습니다.

## 수정이 된 코드는?

기존의 코드는 accessToken과 refreshToken이 모두 만료됐을 경우에 401이 내려가는 문제가 있었는데 이를 방지하기 위해 401이 떨어지면 항상 쿠키를 지우게끔 수정되었습니다.

~~~java
public class RefreshTokenFilter extends GenericFilterBean {

  private final JwtTokenProvider jwtTokenProvider;
  private final RedisUtil redisUtil;
  private final AuthCookieService authCookieService;

  @Override
  public void doFilter(ServletRequest request, ServletResponse response, FilterChain filterChain)
      throws IOException, ServletException {
    HttpServletRequest httpRequest = (HttpServletRequest) request;
    final var accessTokenDto = jwtTokenProvider.tryCheckTokenValid(httpRequest, ACCESS_TOKEN);
    final var refreshTokenDto = jwtTokenProvider.tryCheckTokenValid(httpRequest, REFRESH_TOKEN);
    boolean isAccessExpiredAndRefreshValid = isAccessTokenExpired(accessTokenDto.getResultType()) &&
        isRefreshTokenValid(refreshTokenDto) &&
        isTokenInRedis(refreshTokenDto, httpRequest.getHeader(USER_AGENT));

    if (isAccessExpiredAndRefreshValid) {
      Authentication auth = jwtTokenProvider.getAuthentication(refreshTokenDto.getToken());
      SecurityContextHolder.getContext().setAuthentication(auth);

      String authId = String.valueOf(jwtTokenProvider.getAuthId(refreshTokenDto.getToken()));
      String[] roles = jwtTokenProvider.getRoles(refreshTokenDto.getToken());
      HttpServletResponse httpResponse = (HttpServletResponse) response;

      authCookieService.setNewCookieInResponse(authId, roles, httpRequest.getHeader(USER_AGENT), httpResponse);
    }  else {
      authCookieService.setCookieExpired((HttpServletResponse) response); // Here!
    }

    filterChain.doFilter(request, response);
  }
}
~~~
~~~java
public class AuthCookieService {

  public void setCookieExpired(HttpServletResponse response) { // Here!
    setTokenInCookie(response, "", 0, REFRESH_TOKEN.getTokenName());
    setTokenInCookie(response, "", 0, ACCESS_TOKEN.getTokenName());
  }
}
~~~

하지만 위의 코드를 봐도 로그인 후 쿠키 저장소에 토큰이 저장되지 않는 이유를 알 수 없었습니다. 왜냐하면 쿠키 값을 비우고 로그인을 시도하면 바로 만료되는 쿠키 값을 설정하고 로그인 컴포넌트에서 갓 생성된 토큰 값을 저장해 주기 때문입니다.

~~~java
@Transactional
  public SignInResponse signIn(LoginId loginId, String rawPassword, HttpServletRequest request,
      HttpServletResponse response) {
    Member member = memberRepository.findByProfileLoginId(loginId)
        .orElseThrow(
            () -> new BusinessException(loginId.get(), "loginId", ErrorCode.MEMBER_NOT_FOUND));
    if (member.getProfile().getPassword().isWrongPassword(rawPassword)) {
      throw new BusinessException(loginId.get(), "loginId", ErrorCode.MEMBER_WRONG_ID_OR_PASSWORD);
    }
    authCookieService.setNewCookieInResponse(String.valueOf(member.getId()), // Here!!
        getRoles(member), request.getHeader(HttpHeaders.USER_AGENT), response);
    return SignInResponse.of(member, Arrays.stream(getRoles(member)).toList());
  }
~~~
~~~java
public void setNewCookieInResponse(String authId, String[] roles, String userAgent, HttpServletResponse response) {
    String newRefreshToken = jwtTokenProvider.createAccessToken(REFRESH_TOKEN, authId, roles);
    setTokenInCookie(response, newRefreshToken, (int) REFRESH_TOKEN.getExpiredMillis() / 1000, // refreshToken 저장
        REFRESH_TOKEN.getTokenName());
    String newAccessToken = jwtTokenProvider.createAccessToken(ACCESS_TOKEN, authId, roles);
    setTokenInCookie(response, newAccessToken, (int) REFRESH_TOKEN.getExpiredMillis() / 1000, // accessToken 저장
        ACCESS_TOKEN.getTokenName());
    redisUtil.setDataExpire(JwtTokenProvider.getRefreshTokenKeyForRedis(authId, userAgent), newRefreshToken, REFRESH_TOKEN.getExpiredMillis());
  }
~~~


## 로그인 Request만 보내는 것이 아니다

처음에는 Sign-in Request만 집중해서 살펴봤는데 시야를 넓게 보니 추가적으로 보내는 Request들이 있음을 확인했습니다. 어디로 날리나 확인했더니 Sign-in과 동일하게 백엔드 서버로 Request를 날리고 있음을 확인했습니다. 

![image1](https://github.com/02ggang9/02ggang9.github.io/blob/master/_posts/images/keeper/cookieExpried/images2.png?raw=true)

그렇다면 얘네들도 Spring Security Filter에 걸리게 될 텐데 이 API가 accessToken과 refreshToken을 들고 필터링을 돈다면 어떤 일이 발생하는지 생각해 봤습니다. accessToken과 refreshToken이 정상이여도 else 문으로 빠져 토큰 값을 바로 삭제시킨다는 것을 알아냈습니다. 

~~~java
public class RefreshTokenFilter extends GenericFilterBean {
  @Override
  public void doFilter(ServletRequest request, ServletResponse response, FilterChain filterChain)
      throws IOException, ServletException {
    HttpServletRequest httpRequest = (HttpServletRequest) request;
    final var accessTokenDto = jwtTokenProvider.tryCheckTokenValid(httpRequest, ACCESS_TOKEN);
    final var refreshTokenDto = jwtTokenProvider.tryCheckTokenValid(httpRequest, REFRESH_TOKEN);
    boolean isAccessExpiredAndRefreshValid = isAccessTokenExpired(accessTokenDto.getResultType()) &&
        isRefreshTokenValid(refreshTokenDto) &&
        isTokenInRedis(refreshTokenDto, httpRequest.getHeader(USER_AGENT));

    if (isAccessExpiredAndRefreshValid) {
      Authentication auth = jwtTokenProvider.getAuthentication(refreshTokenDto.getToken());
      SecurityContextHolder.getContext().setAuthentication(auth);

      String authId = String.valueOf(jwtTokenProvider.getAuthId(refreshTokenDto.getToken()));
      String[] roles = jwtTokenProvider.getRoles(refreshTokenDto.getToken());
      HttpServletResponse httpResponse = (HttpServletResponse) response;

      authCookieService.setNewCookieInResponse(authId, roles, httpRequest.getHeader(USER_AGENT), httpResponse);
    }  else { // 이쪽으로 들어감
      authCookieService.setCookieExpired((HttpServletResponse) response); 
    }

    filterChain.doFilter(request, response);
  }
}
~~~

이를 해결하기 위해서는 accessToken과 refreshToken이 정상이라면 바로 다음 필터로 넘어가게 수정해야 했습니다. 따라서 else if 문으로 두 토큰 값이 Expired 일 때만 토큰 값을 삭제하도록 수정했습니다. 이렇게 하면 로그인 후 accessToken과 refreshToken을 발급받고 바로 소멸되는 문제를 방지할 수 있습니다.

~~~java
public class RefreshTokenFilter extends GenericFilterBean {
  @Override
  public void doFilter(ServletRequest request, ServletResponse response, FilterChain filterChain)
      throws IOException, ServletException {

    ...

    if (isAccessExpiredAndRefreshValid) {
      Authentication auth = jwtTokenProvider.getAuthentication(refreshTokenDto.getToken());
      SecurityContextHolder.getContext().setAuthentication(auth);

      String authId = String.valueOf(jwtTokenProvider.getAuthId(refreshTokenDto.getToken()));
      String[] roles = jwtTokenProvider.getRoles(refreshTokenDto.getToken());
      HttpServletResponse httpResponse = (HttpServletResponse) response;

      authCookieService.setNewCookieInResponse(authId, roles, httpRequest.getHeader(USER_AGENT), httpResponse);
    } else if (accessTokenDto.getResultType() == EXPIRED && refreshTokenDto.getResultType() == EXPIRED) {
      authCookieService.setCookieExpired((HttpServletResponse) response);
    }

    filterChain.doFilter(request, response);
  }
}
~~~

## 로깅은 중요하다

위 코드로 수정하고 Prod 서버에 배포할 때 "이제 진짜 해결이다!!" 생각을 하면서 들떠있었는데 막상 테스트를 해 보니 똑같이 토큰 값이 바로 만료되버렸습니다. 왜 이럴까 곰곰히 생각해 봤는데 Token 값이 없으면 Expried 타입이라고 예상했지만 실은 다른 것일 수도 있겠다고 결론이 났습니다. 

~~~java
@Override
  public void doFilter(ServletRequest request, ServletResponse response, FilterChain filterChain)
      throws IOException, ServletException {

    ...

    if (tokenValidationDto.isValid()) {
      Authentication auth = jwtTokenProvider.getAuthentication(tokenValidationDto.getToken());
      SecurityContextHolder.getContext().setAuthentication(auth);
    } else {
      log.info(tokenValidationDto.getResultType().getMsg()); // Here!!
    }

    filterChain.doFilter(request, response);
  }
~~~

위의 코드는 다음 Filter에 있는 코드인데 토큰 값이 유효하지 않다면 로그를 찍고 다음 필터로 넘어갑니다. 이 로그에 찍힌 값을 확인하면 문제를 해결할 수 있겠다고 생각했고 Prod 서버에 들어가려고 했는데 양도 계약서를 작성할 때 운영 Prod에는 접근하지 않겠다고 사인을 해 버린 기억이 떠올랐습니다. 그래서 실제 로그를 확인할 수 있는 방법이 없었지만 얼른 로컬에서 도커를 말아 PostMan으로 확인했고 결과는 UNKNOWN 타입인 것을 확인했습니다. 

![image1](https://github.com/02ggang9/02ggang9.github.io/blob/master/_posts/images/keeper/cookieExpried/image3.png?raw=true)

이 로그를 찍는 한 줄이 없었다면 5가지의 경우를 확인하기 위해 Prod 서버에 5번 배포하는 일이 생겼을 것입니다.

## 개선해야 할 점

### Dev 서버에 배포가 안되는 문제를 해결

위의 글에서 Prod 서버에서 테스트 하는 모습을 보고 기겁하셨을 수도 있습니다. 사실 저희도 Dev 서버에서 테스트를 하고 싶었지만 3개월 전부터 Dev 서버 자동배포가 되지 않는다는 사실을 깨달았습니다. 이 때문에 핫픽스된 코드를 Prod 서버에서 테스트할 수 밖에 없었고 하루라도 빨리 자동배포가 안되는 문제를 해결해야겠다고 생각했습니다.

### else if문 줄이기

너무 긴급한 상황이라 else if 문으로 대응을 해버렸습니다. 또 다른 경우의 수도 생각하지 못해 불안한 감이 있습니다. 급한 불은 껐으니까 다음 회의부터 수정하도록 하겠습니다.

### CICD 코드 수정

CICD가 자동으로 트리거되는 것이 편하다고 생각했는데 CI/CD로 분리하고 수동으로 돌리는 것이 오늘의 불상사를 막을 수 있겠다고 생각했습니다. 이 부분도 회의를 거친 후 수정하도록 하겠습니다.

