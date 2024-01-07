---
published: true
title:  "Spring MVC - 웹 애플리케이션 이해"
categories:
  - spring
---

*글에서 나오는 모든 사진은 스프링 MVC 1편 - 백엔드 웹 개발 핵심 기술, -김영한- 님의 강의에서 캡처했습니다.*

## 웹 서버, 웹 애플리케이션 서버

Web Server는 정적인 리소스(HTML, CSS, 이미지)를 제공할 수 있습니다. 가장 잘 알려져 있는 예시로는 NGINX, APACHE가 있습니다. 반면에 Web Application Server(WAS)는 정적인 리로스뿐만 아니라, 동적인 리소스를 제공할 수 있습니다. 대표적인 예시로 Tomcat이 있습니다.

정적인 리소스와 동적인 리소스 둘 다 제공할 수 있는 WAS만 사용하면 다음 그림과 같이 시스템을 구성할 수 있습니다.

![WAS1](https://github.com/02ggang9/02ggang9.github.io/blob/master/_posts/images/spring/mvc/Chapter1/WAS.png?raw=true "출처: 스프링 MVC 1편 - 백엔드 웹 개발 핵심 기술 -김영한-")

하지만 이런 방식은 큰 문제점이 있습니다. WAS가 정적, 동적, 어플리케이션 로직 등 너무 많은 역할을 하고 있기 때문에 서버가 다운될 우려가 있습니다. 또, 비용이 적게드는 정적인 리소스 때문에 비용이 많이 드는 어플리케이션 로직을 수행하는데 어려움을 겪을 수 있습니다. 이러한 문제는 WAS 앞단에 Web Server를 둬 해결할 수 있습니다.

![WAS2](https://github.com/02ggang9/02ggang9.github.io/blob/master/_posts/images/spring/mvc/Chapter1/WAS2.png?raw=true "출처: 스프링 MVC 1편 - 백엔드 웹 개발 핵심 기술 -김영한-")

이런 구조로 된다면 WAS의 역할이 줄어들고 어플리케이션 로직에만 집중할 수 있게 됩니다. 그리고 정적인 리소스를 많이 제공해야 한다면 Web Server만 증설하여 문제를 해결할 수 있고, 동적인 리소스를 많이 제공해야 한다면 WAS를 증설하여 문제를 해결할 수 있습니다.

## 서블릿

서블릿은 클라이언트의 요청 처리와 응답 반환을 쉽게 도와주는 자바 웹 프로그래밍 기술입니다. 서블릿이 없다면 아래 사진처럼 번거러운 작업을 개발자가 하나하나 해야합니다. 서블릿을 사용한다면 쉽게 클라이언트의 요청에 담긴 메시지를 꺼내올 수 있고 쉽게 클라이언트에게 응답을 보낼 수 있습니다.

스프링 부트에 있는 임베디드 톰캣(WAS)은 서블릿을 지원하는데 아래와 같은 코드로 서블릿을 등록하고, 사용할 수 있습니다.

~~~java
@ServletComponentScan // 자동 서블릿 등록 (동등, 하위 패키지를 다 뒤짐)
@SpringBootApplication
public class ServletApplication {

	public static void main(String[] args) {
		SpringApplication.run(ServletApplication.class, args);
	}
}
~~~

~~~java
@WebServlet(name = "helloServlet", urlPatterns = "/hello")
public class HelloServlet extends HttpServlet {

    @Override
    protected void service(HttpServletRequest req, HttpServletResponse resp) throws ServletException, IOException {
        System.out.println("HelloServlet.service");

        System.out.println("req = " + req);
        System.out.println("resp = " + resp);

        String username = req.getParameter("username");
        System.out.println("username = " + username);

        resp.setContentType("text/plain");
        resp.setCharacterEncoding("utf-8");
        resp.getWriter().write("hello " + username);
    }
}

~~~

![Servlet](https://github.com/02ggang9/02ggang9.github.io/blob/master/_posts/images/spring/mvc/Chapter1/서블릿2.png?raw=true "출처: 스프링 MVC 1편 - 백엔드 웹 개발 핵심 기술 -김영한-")

WAS는 클라이언트가 보낸 메시지를 통해 Request와 Response 객체를 생성하고 적절한 서블릿을 실행시킵니다. 서블릿 객체는 싱글톤으로 관리되므로 공유 변수 사용에 각별히 주의해야 합니다.


## 쓰레드

하나의 메인 쓰레드만 가지고 클라이언트의 요청을 처리하게 되면 여러가지 문제가 발생할 수 있습니다. 기본적으로 여러 클라이언트의 요청을 처리하는 속도가 느리고, Servlet에서 처리가 지연되면 그대로 장애로 이어질 수 있습니다. 이는 클라이언트 요청마다 쓰레드를 생성하는 방법으로 해결할 수 있습니다. 하지만 이 방법 또한 여러가지 문제점이 있습니다. 클라이언트의 요청이 급격하게 늘어난다면 메모리와 CPU 점유율이 한계치에 도달하고 WAS가 다운될 수 있습니다.

최종적으로는 WAS는 쓰레드 풀에 미리 쓰레드를 생성하고 클라이언트의 요청이 들어올때마다 쓰레드를 가져다가 사용하고 반납하는 방법을 사용합니다. 

![Thread](https://github.com/02ggang9/02ggang9.github.io/blob/master/_posts/images/spring/mvc/Chapter1/쓰레드1.png?raw=true "출처: 스프링 MVC 1편 - 백엔드 웹 개발 핵심 기술 -김영한-")

성능이 안나온다면 클라우드 환경일 경우 단순 서버를 증설해서 해결할 수 있지만 온프레미스라면 적절한 쓰레드 풀 수를 찾아 성능을 튜닝할 수 밖에 없습니다. 대표적인 성능 측정 툴은 nGrinder, 제이미터 등이 있지만 nGrinder는 가장 추천하신다고 하셨습니다.

