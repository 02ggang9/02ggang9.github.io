트러블 슈팅

Refused to apply style from 'http://localhost:8080/login' because its MIME type ('text/html') is not a supported stylesheet MIME type, and strict MIME checking is enabled.

스프링 시큐리티는 기본적으로 css, js html 같은 파일도 다 인증을 검사함. 그 기능을 꺼주지 않으면 css를 참고할 수 없는 문제가 발생해서 포맷팅이 엉망으로 출력됨 ㄷ; 아래 추가하면 됨

~~~java
    @Bean
    public WebSecurityCustomizer webSecurityCustomizer() {
        return web -> web.ignoring().requestMatchers("/**");
    }

~~~