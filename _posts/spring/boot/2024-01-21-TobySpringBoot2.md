---
published: true
title:  "Spring Boot - 스프링 부트란 무엇인가?"
categories:
  - spring
---

~~~java
public class MySpringMvcApplication {

	public static void main(String[] args) {
        
	}

}
~~~

~~~java
public class MySpringMvcApplication {

	public static void main(String[] args) {
		TomcatServletWebServerFactory serverFactory = new TomcatServletWebServerFactory();
		WebServer webServer = serverFactory.getWebServer(servletContext -> {
			servletContext.addServlet("hello", new HttpServlet() {
				@Override
				protected void service(HttpServletRequest req, HttpServletResponse resp)
						throws ServletException, IOException {
					String name = req.getParameter("name");

					resp.setStatus(HttpStatus.OK.value());
					resp.setHeader(HttpHeaders.CONTENT_TYPE, MediaType.TEXT_PLAIN_VALUE);
					resp.getWriter().println("Hello " + name);
				}
			}).addMapping("/hello");
		});

		webServer.start();
	}

}

~~~