---
published: false
title:  "Spring - Proxy와 데코레이터 패턴 및 프록시 패턴"
categories:
  - spring
---

## 스프링 부트
BOOT, 부팅
최소한의 인간 개입으로 시작하고 완전히 작동하는 것을 의미. 예를 들어 컴퓨터 전원을 누르면 우린 부팅이라고 하는데 버튼하나 띡 눌렀는데 컴퓨터가 완전히 동작한다.

시작을 위한 복잡한 설정을 스프링 부트가 해결해준다! 

스프링을 편리하게 사용할 수 있도록 지원, 최근에는 기본으로 사용한다. 관례에 의한 간결한 설정. 단독으로 실행할 수 있는 스프링 애플리케이션을 쉽게 생성한다.

스프링 부트 핵심 기능 5가지
WAS: Tomcat 같은 웹 서버를 내장해서 별도의 웹 서버를 설치하지 않아도 됨.

라이브러리 관리
손쉬운 빌드 구성을 위한 스타터 종속성 제공
웹이라는 라이브러리를 땡기면 걔가 필요한 라이브러리를 다 땡겨와 주는거임 starter
라이브러리의 버전을 자동으로 관리
충돌이 날 수 있는데 버전을 자동으로 관리해줌. 옛날에는 버전을 다 지정해줬는데 부트를 쓰면 알아서 라이브러리 버전을 다 맞춰줌. 그리고 호환성을 스프링 부트에서 검증을 다 해놓음;;

자동구성:
프로젝트 시작에 필요한 스프링과 외부 라이브러리의 빈을 자동으로 등록해줌.

외부설정:
환경에 따라 달라져야 한느 외부 설정 공통화

프로덕션 준비:
모니터링을 위한 메트릭, 상태 확인 기능 제공

스프링 프레임워크, 스프링 부트 -> 필수
스프링 데이터, 스프링 세션, 시큐리티, Rest Docs, 배치 등등 -> 선택

"스프링 부트는 스프링 프레임워크를 쉽게 사용할 수 있게 도와주는 도구일 뿐이다!"

왜 배워야 함?
부트는 편리하지만, 너무 많은 것을 자동화 해줌
최소한 스프링 부트가 어떤 원리로 작동하는지 알아두어야 함 -> 그래야 문제가 발생했을 때 해결 가능
스프링 부트의 원리를 이해하면 문제점을 쉽게 파악

스프링 부트는 수 많은 편의 기능들을 제공
대부분의 개발자가 비슷비슷하게 고만고만하게 고민하는 기능을 부트가 이미 만들어서 제공
액츄에이터를 통한 모니터링 관리 기능, 개발 시간 단축


## 웹 서버와 서블릿 컨테이너

### 웹 서버와 스프링 부트 소개
먼저 서버에 WAS를 설치해야 힘; WAS에서 동작하도록 서블릿 스펙에 맞추어 코드를 작성하고 WAR 형식으로 빌드해서 war 파일을 만듬. 그리고 war 파일을 

## 톰캣 설치
1. 아래 링크로 가서 다운 zip 파일을 다운하면 됨
2. chmod 755 * 
3. ./startup.sh -> 후 localhost8080으로 이동

## WAR 빌드와 배포
자바 위에 웹 어플리케이션 서버가 실행되고 그 위에 WAR가 올라가서 실행된다.


# 액츄에이터
치킨 주문하기 했는데 빡침. 모니터링은 운영 단계에서 아주 중요하다!

고객센터에서 전화오면 개발자는 부끄러워야 함. 이미 고객들이 많이 피해를 보고 전화하는 그 단계임

지표 -> CPU 사용양
추적 -> 애플리케이션 코드가 어디서 문제가 터지는지
감사 -> 로그인 로그아웃 하는 이력들을 남겨서 추적하는 것
모니터링 -> 전체적으로 지표를 모니터링 함


# 마이크로피터, 프로메테우스, 그라파나

## 마이크로미터
"마이크로피터는 애플리케이션 메트릭 파사드라고 불리는데, 애플리케이션의 메트릭을 마이크로미터가 정한 표준 방법으로 모아서 제공해준다. 쉽게 말해서 마이크로미터가 추상화를 통해서 구현체를 쉽게 갈아끼울 수 있도록 해둠"

스프링 부트 액츄에이터는 마이크로미터를 기본으로 내장해서 사용해서 걱정할 필요 없다.

~~~java
implementation 'orgspringframeworkboot:spring-boot-starter-actuator' //
actuator 추가
~~~

기본으로 다양한 메트릭을 제공하는데 그 수는 너무 많다. 그래서 필요할 때마다 찾으면 될 것 같다.

http://localhost:8080/actuator

위의 경로로 들어가면 확인할 수 있다.
(엑츄에이터, 마이크로미터 아키텍처 사진)


## 프로메테우스 
애플리케이션에서 발생한 그 순간의 메트릭을 확인하고 보관하는 DB이다.
(프로메테우스 아키텍처 사진)

~~~java
implementation 'io.micrometer:micrometer-registry-prometheus' //추가

http://localhost:8080/actuator/prometheus
~~~

모든 메트릭이 프로메테우스 포멧으로 바뀐다.

얼마나 주기적으로 메트륵을 수집할 것인지 아래의 yml 파일로 설정할 수 있다.

~~~yml

~~~

http://localhost:9090/targets
위에서 연동이 잘 되었는지 확인


## 그라파나
프로메테우스에 저장된 메트릭 데이터를 불러서 수 많은 그래프를 제공한다.
(그라파나 아키텍처 사진)

http://localhost:3000
admin/admin

~~~yml
version: '3.5'

services:
  app:
    container_name: app2
    image: keeper-homepage-app:build
    entrypoint: bash -c "chmod +x /entrypoint.sh && /entrypoint.sh"
    environment:
      - SECRET=${SECRET}
      - MAIL_HOST=${MAIL_HOST}
      - MAIL_PORT=${MAIL_PORT}
      - MAIL_USERNAME=${MAIL_USERNAME}
      - MAIL_PASSWORD=${MAIL_PASSWORD}
      - MYSQL_DATABASE=${MYSQL_DATABASE}
      - MYSQL_USER=${MYSQL_USER}
      - MYSQL_PASSWORD=${MYSQL_PASSWORD}
    ports:
      - 8082:8080
    depends_on:
      - db

  db:
    container_name: db2
    image: keeper-homepage-db:build
    environment:
      - MYSQL_DATABASE=${MYSQL_DATABASE}
      - MYSQL_ROOT_PASSWORD=${MYSQL_ROOT_PASSWORD}
      - MYSQL_USER=${MYSQL_USER}
      - MYSQL_PASSWORD=${MYSQL_PASSWORD}
    command:
      - --character-set-server=utf8mb4
      - --collation-server=utf8mb4_unicode_ci
    ports:
      - 3306:3306

  redis:
    container_name: redis2
    image: redis:6.2
    ports:
      - 6379:6379

  nginx:
    container_name: nginx2
    image: keeper-homepage-front:build
    volumes:
      - ./docker/default.conf:/etc/nginx/conf.d/default.conf
    ports:
      - 8081:80
  
  prometheus:
    image: prom/prometheus
    container_name: prometheus
    volumes:
      - ./prometheus.yml:/prometheus/prometheus.yml
    ports:
      - 9090:9090
    restart: always
  
  grafana:
    image: grafana/grafana
    container_name: grafana
    ports:
      - 3000:3000
    volumes:
      - ./grafana/grafana-volume:/var/lib/grafana
    restart: always
~~~

엑츄에이터는 들어가 있드라고요? 프로메테우스는 넣어줬슴.

~~~gradle
dependencies {
    implementation 'org.springframework.boot:spring-boot-starter-web'
    implementation 'org.springframework.boot:spring-boot-starter-websocket'
    implementation 'io.micrometer:micrometer-registry-prometheus' //추가
    implementation 'org.springframework.boot:spring-boot-starter-validation'
}
~~~

하고 R1의 application.properties에서 management 추가 2줄

~~~yml
management.endpoints.web.exposure.include= "*"
management.endpoint.health.show-details=always
~~~



https://www.inflearn.com/questions/1030769/docker%EB%A1%9C-prometheus-grafana-%EC%82%AC%EC%9A%A9%ED%95%98%EB%8A%94-%EA%B2%BD%EC%9A%B0-%EC%84%A4%EC%A0%95-%EA%B0%80%EC%9D%B4%EB%93%9C


~~~yml
version: '3.5'

services:
  app:
    container_name: app2
    image: keeper-homepage-app2:build
    entrypoint: bash -c "chmod +x /entrypoint.sh && /entrypoint.sh"
    environment:
      - SECRET=${SECRET}
      - MAIL_HOST=${MAIL_HOST}
      - MAIL_PORT=${MAIL_PORT}
      - MAIL_USERNAME=${MAIL_USERNAME}
      - MAIL_PASSWORD=${MAIL_PASSWORD}
      - MYSQL_DATABASE=${MYSQL_DATABASE}
      - MYSQL_USER=${MYSQL_USER}
      - MYSQL_PASSWORD=${MYSQL_PASSWORD}
    ports:
      - 8082:8080
    depends_on:
      - db

  db:
    container_name: db2
    image: keeper-homepage-db:build
    environment:
      - MYSQL_DATABASE=${MYSQL_DATABASE}
      - MYSQL_ROOT_PASSWORD=${MYSQL_ROOT_PASSWORD}
      - MYSQL_USER=${MYSQL_USER}
      - MYSQL_PASSWORD=${MYSQL_PASSWORD}
    command:
      - --character-set-server=utf8mb4
      - --collation-server=utf8mb4_unicode_ci
    ports:
      - 3306:3306

  redis:
    container_name: redis2
    image: redis:6.2
    ports:
      - 6379:6379

  nginx:
    container_name: nginx2
    image: keeper-homepage-front:build
    volumes:
      - ./docker/default.conf:/etc/nginx/conf.d/default.conf
    ports:
      - 8081:80
  
  prometheus:
    image: prom/prometheus
    container_name: prometheus
    volumes:
      - ./prometheus/config:/etc/prometheus/
      - prometheus-data:/prometheus
    ports:
      - 9090:9090
    command: # web.enalbe-lifecycle은 api 재시작없이 설정파일들을 reload 할 수 있게 해줌
      - '--config.file=/etc/prometheus/prometheus.yml'
      - '--storage.tsdb.path=/prometheus'
    restart: always
    networks:
      - goodjob-network
  
  grafana:
    image: grafana/grafana
    container_name: grafana
    # user: "$GRA_UID:$GRA_GID"
    ports:
      - 3000:3000
    volumes:
      - ./grafana/grafana-volume:/var/lib/grafana
    restart: always
    networks:
      - goodjob-network

volumes:
  grafana-data:
  prometheus-data:

networks:
  goodjob-network:
    driver: bridge
~~~