---
published: false
title:  "Spring Boot - 스프링 부트란 무엇인가?"
categories:
  - spring
---

## 스프링 부트란 무엇인가

> Spring Boot makes it easy to create stand-alone, production-grade Spring based Applications that you can “just run”

스프링 부트는 스프링 기반으로 실무 환경에 사용할 수 있는 수준의 독립 실행형 애플리케이션을 쉽게 만들어 준다고 공식 사이트에 기재되어 있습니다. 정확한 개념(스프링 부트는 00이다)는 알려주지 않고 행위에 대해서만 알려주고 있습니다. 제가 생각하기엔 스프링 부트는 쉽게 스프링 기반의 독립 실행형 애플리케이션을 만들어 주는 도구라고 생각합니다.

### Makes it easy

과거 Spring은 web.xml 파일을 설정하고 war로 패키징 후 Web Server 설치와 Web Application Server 설치하는 등 다양한 과정을 거쳐야 겨우 hello world라는 간단한 프로그램을 만들 수 있었습니다. 이 과정은 매우 귀찮고 학습 곡선 또한 컸습니다. 하지만 현재는 Spring Boot 덕분에 커맨드 라인 한 줄로 어플리케이션을 쉽게 실행시킬 수 있습니다.

### Spring based Stand-alone application

스프링은 스프링 부트와 다릅니다. 스프링 부트는 스프링 기반의 독립 실행형 애플리케이션을 만들도록 도와주는 Tool이지 둘은 완전히 다릅니다. 독립 실행형 애플리케이션은 main Method를 호출하는 것만으로도 관련한 설정을 해주고 was가 실행되도록 해주는 것입니다.

```java
@SpringBootApplication
public class MySpringMvcApplication {

	public static void main(String[] args) {
		SpringApplication.run(MySpringMvcApplication.class, args);
	}

}
```

## Containerless란 무엇인가

Containerless란 컨테이너 관리를 신경 쓰지 않고 개발하자는 의미입니다. 컨테이너가 필요 없다는 의미는 아닙니다. 앞서 말씀드렸듯이 hello world라는 프로그램을 실행시키기 위해서는 서블릿 컨테이너도 실행시켜야 하고, 정말 다양한 설정(web.xml)을 해야 합니다. 이러한 과정은 프로젝트 시작할 때 잠깐 사용되고 잊어버리게 됩니다. 또, 톰캣을 사용하지 않고 다른 WAS를 사용하는 집단으로 들어가게 된다면 다른 설정들을 학습해야 합니다.

스프링 부트는 “Just run”하는 것 만으로 WAS(Tomcat)이 실행되고 포트가 열리고, 다양한 설정이 알아서 되어버립니다. 서블릿 컨테이너라는 것은 자바 웹 표준 기술이기 때문에 필요하지만 설치하고 관리하기 위한 지식 습득 없이 일단 스프링 애플리케이션을 만들고 사용할 수 있게 됩니다. 

저는 Spring Boot 3.0 버전에서 부터 개발을 하기 시작했습니다. 인텔리제이의 Run 버튼만 누르면 알아서 서버가 뜨고 8080 포트가 열렸기 때문에 WAS에 대해서 하나도 몰랐지만 바로 프로젝트에 투입될 수 있었습니다. 과거를 생각해 봤을 때 Spring Boot는 정말로 자신의 역할(Containerless)을 충실히 수행한 것 같습니다.

## 결론

스프링 부트는 쉽게 스프링 기반의 독립 실행형 애플리케이션을 만들 수 있도록 도와주는 Tool입니다. 과거에는 간단한 hello world 프로그램을 배포하기 위해서 정말 귀찮은 과정을 반복해야 했습니다. 하지만 스프링 부트는 WAS 설치와 실행, 디폴트 설정을 해 줌으로써 간단한 커맨드 라인 하나만으로 어플리케이션을 실행할 수 있습니다. 

다음 글에서는 스프링 부트 없이 Tomcat을 띄워보고 Servlet을 만든 후 매핑까지 해보도록 하겠습니다.