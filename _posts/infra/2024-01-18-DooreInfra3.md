---
published: true
title:  "Infra - Doo-re Infra(2) - Gradle과 JAR"
categories:
  - infra
---

## 서론

BDD(부산 개발자 소모임)의 Doo-re 프로젝트에서 인프라를 담당하게 되었습니다. BDD는 문서화를 굉장히 중요시하기 때문에 인프라를 구축하면서 필요한 개념과 과정들을 모두 작성하기로 했습니다.

이번 문서에서는 Gradle과 JAR가 무엇인지 알아보도록 하겠습니다.


## Gradle이란

> Gradle Build Tool is a fast, dependable, and adaptable open-source build automation tool with an elegant and extensible declarative build language. [docs.gradle.org](https://docs.gradle.org/current/userguide/userguide.html?_gl=1*1gkwnn3*_ga*Nzg0OTA5NjY5LjE3MDU1NDY4Nzc.*_ga_7W7NC6YNPT*MTcwNTU0Njg3Ny4xLjEuMTcwNTU0Njg5MC40Ny4wLjA.)

Gradle은 아래의 Task를 자동으로 해줍니다.

~~~md
## Compile
Java 파일을 바이트코드로 변환

## Test
Unit Test, Integration 테스트를 지원

## Packaging
.class 파일을 패키징 해 jar or war 파일로 변환
~~~

![gradle1](https://github.com/02ggang9/02ggang9.github.io/blob/master/_posts/images/infra/infra1/gradle1.png?raw=true)

보통 자바 개발자분들은 인텔리제이를 사용하는데 많이 하는 설정이 Build and Run을 Gradle에서 IntelliJ IDEA로 바꿔주는 것 입니다. 저는 매번 Gradle 설정을 바꾸고 IntelliJ의 Run 버튼을 눌러 실행해 위의 3개의 과정을 까먹었는데 여러분들도 이번 기회에 리마인드 하시면 좋을 것 같습니다.




## Jar

JAR는 여러개의 자바 클래스 파일과 메타데이터를 하나의 파일로 모아서 배포하기 위한 패키지 파일입니다. 위의 Gradle Build Tool의 마지막 단계에서 .class 파일을 .jar나 war 파일로 만들어 주는데 파일 경로는 아래와 같습니다.

> your_project_name -> build -> libs -> your_project_name-0.0.1-SNAPSHOT.jar

### SNAPSHOT

조직 내부에서 사용되는 라이브러리는 SNAPSHOT 버전으로 만들어서 관리한다고 합니다. SNAPSHOT 버전을 확인하는 방법은 build.gralde에서 확인하실 수 있습니다.

~~~java
plugins {
	id 'java'
	id 'org.springframework.boot' version '3.2.1'
	id 'io.spring.dependency-management' version '1.1.4'
}

group = 'io.springbatch'
version = '0.0.1-SNAPSHOT' // Here !!

java {
	sourceCompatibility = '17'
}

... (dependencies) ... 
~~~

위의 version 부분이 변경되지 않으면 빌드를 많이 해도 항상 your_project_name-0.0.1-SNAPSHOT 버전이므로 뒤에서 작성할 Dockfile에서 혼동을 겪지 않으셔도 됩니다.

## JAR 실행

Gradle이 만들어준 .jar 파일을 실행시켜야 Spring Boot가 동작하고 정상적으로 Embedder Tomcat이 실행됩니다. WAS가 떠야지 외부로부터 접근이 가능하므로 수동 배포의 최종 종착지는 JAR 파일을 실행하는 것 입니다. 아래는 인텔리제이의 Run 버튼을 누르지 않고 Spring Boot를 띄우는 방법이고 익숙해 져야 합니다.

~~~java
java -jar your_project_name-0.0.1-SNAPSHOT.jar

// 예시

java -jar spring-batch-0.0.1-SNAPSHOT.jar
~~~

![jar1](https://github.com/02ggang9/02ggang9.github.io/blob/master/_posts/images/infra/infra1/jar1.png?raw=true)

실제로 인텔리제이의 Run 버튼을 누른 후 Console창을 확인하면 아래의 코드가 생량되어 있는 것을 확인하실 수 있습니다. 

![jar1](https://github.com/02ggang9/02ggang9.github.io/blob/master/_posts/images/infra/infra1/jar2.png?raw=true)

생략된 부분을 펼치면 아래의 코드처럼 수 많은 .jar를 실행시키는 것을 확인하실 수 있습니다. (인텔리제이는 java -jar 대신 -classpatch 옵션과 명시적으로 디렉터리를 지정해주는 방식을 사용하고 있습니다)

~~~java
/Library/Java/JavaVirtualMachines/jdk-20.jdk/Contents/Home/bin/java -XX:TieredStopAtLevel=1 
-Dspring.output.ansi.enabled=always 
-Dcom.sun.management.jmxremote 
-Dspring.jmx.enabled=true 
-Dspring.liveBeansView.mbeanDomain 
-Dspring.application.admin.enabled=true 
-Dmanagement.endpoints.jmx.exposure.include=* 
-javaagent:/Applications/IntelliJ IDEA.app/Contents/lib/idea_rt.jar=49340:/Applications/IntelliJ IDEA.app/Contents/bin 
-Dfile.encoding=UTF-8 
-Dsun.stdout.encoding=UTF-8 
-Dsun.stderr.encoding=UTF-8 
-classpath /Users/leesoobeen/Desktop/인프런 강의/스프링배치/spring-batch/build/classes/java/main:/Users/leesoobeen/Desktop/인프런 강의/스프링배치/spring-batch/build/resources/main:/Users/leesoobeen/.gradle/caches/modules-2/files-2.1/org.projectlombok/lombok/1.18.30/f195ee86e6c896ea47a1d39defbe20eb59cd149d/lombok-1.18.30.jar
... (엄청 많은 .jar를 실행) ...
~~~


## 결론

Gradle은 자동 Build Tool입니다. 바이트 코드 변환 후 테스트 검증 및 JAR 파일로 압축하는 과정을 거칩니다. 평소 인텔리제이의 Run 버튼을 눌러 어플리케이션을 실행시키지만 java의 jar 명령어로도 어플리케이션을 실행시킬 수 있습니다. 도커는 GUI 환경이 아니기 때문에 CLI로 실행시킬 수 있어야 합니다.
