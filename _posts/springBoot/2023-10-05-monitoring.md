---
published: true
title:  "Spring Boot - 웹 애플리케이션 운영 중 모니터링"
categories:
  - spring
---

현재 KEEPER R2 프로젝트는 운영 단계에 돌입하기 전입니다. 프로메테우스와 그라파나를 활용해서 모니터링을 하면 이상 징후를 사전에 탐지할 수 있고 재빠른 대응이 가능할거라 기대합니다. 그리고 해결하지 못한 @BatchSize 버그때문에 성능을 가늠하기 어려워 한번쯤은 테스트를 해봐야 한다고 생각했습니다. 또, 겨울방학에 이루어질 프로젝트에서도 모니터링 계획이 있기 때문에 선행학습을 하면 도움이 될 것이라 생각해 학습하게 되었습니다.

프로메테우스, 그라파나는 책이 있을 정도로 양이 많습니다. 따라서 포스팅에 다 담을 수 없고 간단한 개념에 대해서 설명하고 KEEPER R1 프로젝트에 프로메테우스와 그라파나를 붙이는 과정과 결과를 정리하겠습니다.

## 마이크로미터, 프로메테우스, 그라파나 개념 및 구조

### 마이크로미터
> 마이크로미터는 애플리케이션 메트릭 파사드라고 불리는데, 애플리케이션의 메트릭(측정 지표)을 마이크로미터가 정한 표준 방법으로 모아서 제공해준다. (실전! 스프링 부트 강의 PPT 4쪽, 김영한)

프로메테우스 뿐만 아니라 다른 모니터링 툴도 있기 때문에 표준 방식으로 측정한 뒤 구현체에 맞춰 변환합니다.

![마이크로미터](https://github.com/02ggang9/02ggang9.github.io/blob/master/_posts/images/springBoot/monitoring/micro.png?raw=true)

### 프로메테우스 및 그라파나
프로메테우스는 앞서 말한 마이크로미터 표준 측정 방식으로 데이터를 수집하고 그라파나는 프로메테우스에서 수집한 데이터를 대쉬보드를 활용해 그래프 등으로 보여줍니다. 프로메테우스는 순간 순간 메트릭을 수집하고 저장하는 DB라고 생각하시면 됩니다.

![모니터링툴아키텍처](https://github.com/02ggang9/02ggang9.github.io/blob/master/_posts/images/springBoot/monitoring/archi.png?raw=true)

## KEEPER R1 프로젝트에서 프로메테우스, 그라파나를 붙이는 과정
KEEPER R1 프로젝트에서는 엑츄에이터를 통한 메트릭 수집은 하고 있었으나 프로메테우스와 그라파나와 같은 모니터링 툴은 사용하지 않았습니다. 그래서 프로메테우스, 그라파나 의존성을 추가하고 작년 겨울 방학 인프라 스터디에서 사용한 도커 컴포즈에 살짝만 코드를 수정해 금방 붙일 수 있었습니다.

### 의존성 및 application.properties 추가
~~~java
implementation 'io.micrometer:micrometer-registry-prometheus' //추가
implementation 'org.springframework.boot:spring-boot-starter-actuator'
runtimeOnly 'io.micrometer:micrometer-registry-prometheus'
~~~

~~~yml
management.endpoints.web.exposure.include=*
management.endpoint.health.show-details=always
~~~

### docker-compose 추가
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

~~~yml
# 프로메테우스 환경 설정 파일

global:
  scrape_interval: 1s # 1초마다 수집
  scrape_timeout: 1s
  evaluation_interval: 15s
alerting:
  alertmanagers:
    - static_configs:
        - targets:
rule_files:
scrape_configs:
  - job_name: "spring-actuator"
    scrape_interval: 1s
    metrics_path: "/actuator/prometheus" # 마이크로미터 표준 측정 방식에서 프로메테우스 방식으로 변경한 메트릭을 수집
    static_configs:
      - targets: ['host.docker.internal:8082'] #localhost:8082로 설정하면 메트릭 수집이 불가

  - job_name: "prometheus"
    static_configs:
      - targets: ['localhost:9090']
~~~

### 결과 사진 (프로메테우스)
![프로메테우스Config](https://github.com/02ggang9/02ggang9.github.io/blob/master/_posts/images/springBoot/monitoring/proConfig.png?raw=true)

![프로메테우스Targers](https://github.com/02ggang9/02ggang9.github.io/blob/master/_posts/images/springBoot/monitoring/proTargets.png?raw=true)

### 결과 사진 (그라파나)
![그라파나DashBoard](https://github.com/02ggang9/02ggang9.github.io/blob/master/_posts/images/springBoot/monitoring/graSpringDash.png?raw=true)

### 결과 사진 (Slack 알림 설정)
![그라파나Slack](https://github.com/02ggang9/02ggang9.github.io/blob/master/_posts/images/springBoot/monitoring/graAlert.png?raw=true)

## 마치며
이번 포스팅에서는 간단하게 모니터링 툴에 대한 개념과 구조에 대해서 살펴봤고 실제 프로젝트에 붙여보는 과정과 결과, 간단한 Slack 알림에 대해서 알아봤습니다. 항상 큰 도움을 받고 있는 선배님께서 올려주신 카카오 유튜브 영상<sup>[1](#1)</sup> 에서도 그라파나를 통해 대규모 트래픽에 대응했습니다. 모니터링 툴과 Slack 연동을 통해서 장애와 이상징후에 대해서 발빠른 대처가 가능할거라 기대합니다.

## 참고자료
<a name="1">[1]</a> 1cm^2속 보이지 않는 카카오의 노력 URL : https://www.youtube.com/watch?v=ro_6M5qc604