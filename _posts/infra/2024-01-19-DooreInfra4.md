---
published: true
title:  "Infra - Doo-re Infra(3) - Infra Code 작성 및 로컬 환경에서 테스트"
categories:
  - infra
---

## 서론

BDD(부산 개발자 소모임)의 Doo-re 프로젝트에서 인프라를 담당하게 되었습니다. BDD는 문서화를 굉장히 중요시하기 때문에 인프라를 구축하면서 필요한 개념과 과정들을 모두 작성하기로 했습니다.

이번 문서에서는 수동 배포를 위한 모든 인프라 코드를 작성해보고 로컬 환경에서 테스트 해보도록 하겠습니다. 아래의 코드는 [doo-re-infra repo](https://github.com/02ggang9/01-doo-re-infrastructure)에서 확인하실 수 있습니다.


## Dockerfile 문법 및 작성

~~~dockerfile
# build/docker/app.dockerfile
# gradle:8.5.0-jdk17 베이스 이미지 파일로 설정
FROM gradle:8.5.0-jdk17 

# cd /home/doo-re (만약 없을 경우 생성)
WORKDIR /home/doo-re

# 커맨드 실행 (즉, /home/doo-re 디렉토리에 백엔드 코드를 git clone)
RUN git clone https://github.com/BDD-CLUB/01-doo-re-back.git .

# 환경 변수 설정(한글 지원을 위한 설정)
ENV LANG=ko_KR.UTF-8 \
    LANGUAGE=ko_KR.UTF-8 \
    LC_ALL=ko_KR.UTF-8

# app.dockerfile 파일 기준으로 docker/data/entrypoint.sh에 있는 쉘 코드를 /home/doo-re/entrypoint.sh에 복사
# app.dockerfile 파일 기준으로 docker/data/application.yml에 있는 쉘 코드를 /home/doo-re/application.yml에 복사
COPY docker/data/entrypoint.sh /entrypoint.sh
COPY docker/data/application.yml /application.yml

~~~

도커 파일의 주된 역할은 컨테이너 실행을 위한 파일을 가져오고 환경 변수를 설정하는 것 입니다. 물론 ENTRYPOINT 명령어를 사용해 쉘 스크립트를 실행시킬 수 있지만 그 과정은 docker-compose 파일에 작성하도록 하겠습니다.

~~~dockerfile
# build/docker/db.dockerfile
# mysql/mysql-server:8.0 베이스 이미지를 사용
FROM mysql/mysql-server:8.0

COPY docker/data/mysql.cnf /etc/mysql/conf.d/mysql.cnf
# COPY docker/data/init.sql /docker-entrypoint-initdb.d/init.sql

RUN ln -sf /usr/share/zoneinfo/Asia/Seoul /etc/localtime

ENV LC_ALL=C.UTF-8
~~~

docker에서 Table 초기화를 하려면 /docker-entrypoint-initdb.d 디렉터리에 init.sql 파일을 삽입하면 됩니다. 하지만 doo-re 프로젝트에서는 schema.sql을 사용해 데이터베이스를 초기화 하는 과정을 거치기 때문에 필요 없습니다. 자세한 사항은 [이 블로그](https://kha0213.github.io/spring/schema.sql/)에서 확인하실 수 있습니다.

## application.yml 파일 작성

~~~yml
# build/docker/data/application.yml

spring:
  datasource:
    url: jdbc:mysql://db:3306/doore?serverTimezone=Asia/Seoul&characterEncoding=UTF-8
    username: ${MYSQL_USER}
    password: ${MYSQL_PASSWORD}
    driver-class-name: com.mysql.cj.jdbc.Driver
  sql:
    init:
      mode: always
  jpa:
    hibernate:
      ddl-auto: none
    properties:
      hibernate:
        format_sql: true
        dialect: org.hibernate.dialect.MySQL8Dialect
    show-sql: true
~~~

위의 yml파일에서 주의 깊게 봐야할 점은 환경 변수를 사용했다는 것 입니다. 로컬 환경에서 테스트할 때는 DB의 id, password를 아무렇게나 설정해도 되지만 dev, prod 서버의 데이터베이스는 관리자만 알고 있어야 합니다. 인프라 코드에서는 환경 설정 파일(.env)와 환경 변수를 설정해 안전하게 깃허브에서 코드를 관리할 수 있습니다.


## entrypoint.sh 작성

~~~sh
# build/docker/data/entrypoint.sh
#!/bin/sh

cp /application.yml /home/doo-re/src/main/resources/application.yml

cd /home/doo-re

gradle build --daemon --build-cache --parallel

java -jar -Duser.timezone=Asia/Seoul /home/doo-re/build/libs/doore-0.0.1-SNAPSHOT.jar

~~~

위의 Shell script는 이미지가 컨테이너화 되고 난 후 실행할 명령어입니다. 위의 코드가 없다면 저희의 app 컨테이너는 실행되고 아무런 동작을 하지 않고 종료하게 됩니다. 이전의 [Infra - Doo-re Infra(2) - Gradle과 JAR](https://02ggang9.github.io/infra/DooreInfra3/)에서 Gradle(빌드 자동화 툴)을 사용해서 자바 기반의 웹 어플리케이션을 빌드했고, Jar 명령어를 통해 CLI에서 스프링 부트를 띄울 수 있는 방법을 배웠습니다. 스프링 부트를 실행한다면 app 컨테이너는 종료하지 않고 계속해서 실행되게 됩니다.

### gradle 옵션

- --daemon: 이 옵션은 Gradle에 데몬이라는 수명이 긴 백 그라운드 프로세스로 실행하도록 합니다. 빌드 실행 속도를 높일 수 있습니다.

- --build-cache: 이 옵션은 빌드 캐싱을 활성화합니다. Gradle은 입력이 변경되지 않은 경우 이전 빌드의 작업 출력을 재사용해 빌드 속도를 크게 향상시킬 수 있습니다.

- --parallel: 이 옵션은 Gradle 작업을 병령로 실행해 빌드 속도를 높일 수 있습니다.

### java 옵션

- -Duser.timezone=Asia/Seoul: JVM에 전달되는 시스템 속성입니다. JVM의 기본 시간대를 서울 시간대로 설정합니다.

위의 시간대 설정을 하지 않으면 잠재적인 위험요소가 많아 필수적으로 해줘야 하는 설정입니다.


## Docker-compose 작성

~~~yml
version: '3.5'

services:
  app:
    container_name: doo-re-app
    image: doo-re-app:build # 컨테이너화 시킬 이미지를 정의할 수 있습니다.
    ports:
      - 8080:8080 # 인바운트 포트와 컨테이너 내부 포트를 매핑할 수 있습니다.
    entrypoint: | # 이미지가 컨테이너화 되고 실행할 명령어를 정의할 수 있습니다.
      bash -c "chmod +x /entrypoint.sh && /entrypoint.sh"
      "/entrypoint.sh"
    environment: # 환경 변수를 .env 파일을 이용해 안전하게 민감 정보를 관리할 수 있습니다.
      - MYSQL_DATABASE=${MYSQL_DATABASE}
      - MYSQL_USER=${MYSQL_USER}
      - MYSQL_PASSWORD=${MYSQL_PASSWORD}
    depends_on: # 아래에서 다루도록 하겠습니다.
      - db

  db:
    container_name: doo-re-db
    image: doo-re-db:build
    environment:
      - MYSQL_DATABASE=${MYSQL_DATABASE}
      - MYSQL_USER=${MYSQL_USER}
      - MYSQL_PASSWORD=${MYSQL_PASSWORD}
      - MYSQL_ROOT_PASSWORD=${MYSQL_ROOT_PASSWORD}
    command: 
      - --character-set-server=utf8mb4
      - --collation-server=utf8mb4_unicode_ci
    ports:
      - 3306:3306

~~~

위의 docker-compose.yml 파일을 통해서 여러 개의 도커 컨테이너를 한번에 띄울 수 있습니다. doo-re 프로젝트나 다른 프로젝트를 띄울 때 주의해야 할 점은 depends_on 입니다. 아래의 사진은 과거 톰 선배님의 인프라 스터디에서 찍은 사진입니다.

![gradle1](https://github.com/02ggang9/02ggang9.github.io/blob/master/_posts/images/infra/infra3/dockercompose.png?raw=true)

사진에서 알 수 있듯이 Gradle은 compile -> teest -> packaging(jar 변환) 과정을 거치게 됩니다. doo-re 프로젝트는 통합 테스트를 포함한 다양한 테스트를 진행합니다. 통합 테스트를 진행할 때는 데이터베이스와 연결이 되어 있어야 하는데 depends_on 설정을 하지 않아 app container가 먼저 띄워져 버리면 테스트 단계에서 실패해 스프링 부트를 실행시킬 수 없습니다. (컨테이너가 실행되는 순서는 알 수 없습니다)

따라서 depends_on 설정을 통해 데이터베이스 컨테이너가 띄워진 후 app 컨테이너가 띄워지고 Gradle을 통한 build 과정을 수행하도록 설계해야 합니다.


## 로컬 환경에서 테스트

~~~sh
# build 폴더로 이동
cd infra/doo-re/build

# app.dockerfile 이미지를 빌드
# 만약 빌드 후 data 폴더의 코드가 수정되었다면 아래의 코드 작성
# docker build --no-cache -t doo-re-app:build -f docker/app.dockerfile .
# 제일 마지막 .은 현재 경로라는 의미이고 현재 경로에서 docker/app.dockerfile의 이미지를 빌드하겠다는 의미입니다.
docker build -t doo-re-app:build -f docker/app.dockerfile .
docker build -t doo-re-db:build -f docker/db.dockerfile .

# 빌드된 docker image 조회
docker images

# docker-compose.yml 파일을 실행해 두 개의 컨테이너를 동시에 실행
# docker-compose up -d 는 백그라운드에서 실행
docker-compose up
~~~

위의 코드를 실행한 결과는 아래와 같습니다. 혹시 빌드가 실패하고 initializationError FAILED doo-re-app java.lang.NoSuchMethodError at SpringExtension.java:381 에러가 터진다면 [이슈](https://github.com/BDD-CLUB/01-doo-re-back/issues/15)에서 해결하실 수 있습니다.


![실행 결과](https://github.com/02ggang9/02ggang9.github.io/blob/master/_posts/images/infra/infra3/local1.png?raw=true)


## 결론

로컬 환경에서 도커 컨테이너를 실행하기 위해 도커 이미지를 포함한 여러 개의 코드를 작성했습니다. 로컬 환경에서 성공했다면 AWS의 EC2에서도 쉽게 성공할 수 있습니다. 

다음은 EC2 인스턴스를 생성하는 방법과 환경 설정 방법에 대해서 알아보도록 하겠습니다.
