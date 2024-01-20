---
published: true
title:  "yml 분리 이유와 caused by: org.yaml.snakeyaml.constructor.duplicatekeyexception at safeconstructor.java 에러 해결"
categories:
  - trouble
---

## 서론

Doo-re 프로젝트에서 인프라를 담당하고 있는데 이상하게 어제까지 잘 됐던 CICD 코드가 작동하지 않았습니다. 아래는 왜 문제가 발생했는지와 해결 방법에 대해서 알아보도록 하겠습니다. 

## 무엇이 문제였나

에러 메시지를 보면 yaml 파일에서 발생한 것을 알 수 있습니다. 또, key 값이 복사되었다고 하는데 이는 같은 키 값이 두 개 이상이라는 의미로 해석할 수 있습니다. 에러 범위를 줄이기 위해 yaml 문법이 맞는지부터 확인했는데 검사 사이트에 돌려봐도 이상이 없었습니다. 그럼 key 값이 중복되었다는 의미인데 아래에서 제가 작성한 yml 파일에서는 그런 모습을 찾을 수 없었습니다.

~~~yaml
spring:
  datasource:
    url: jdbc:mysql://localhost:3306/doore?serverTimezone=Asia/Seoul&characterEncoding=UTF-8
    username: root
    password: 1234
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

답은 CICD 코드에 있었습니다. 아래의 sh 코드는 secrets.APPLICATION_YML 파일을 현재 존재하는 application.yml에 추가하는 코드입니다. 저의 의도는 url과 username과 password가 시크릿 처리된 application.yml을 기존의 파일에 덮어쓰기 한 후 빌드와 테스트를 진행하는 것이였습니다.

~~~yml
- name: Create application.yml
  working-directory: ./src/main/resources
  env:
      APPLICATION_YML: \$ {{ secrets.APPLICATION_YML }}
  run: | # Here!
      echo '\$ {{ secrets.APPLICATION_YML }}' >> application.yml 
~~~

하지만 의도와는 다르게 아래의 application.yml이 탄생되었습니다. 이렇기 때문에 키 값이 중복되었다는 에러 메시지가 발생하였습니다.

~~~yml
spring:
  datasource:
    url: jdbc:mysql://localhost:3306/doore?serverTimezone=Asia/Seoul&characterEncoding=UTF-8
    username: root
    password: 1234
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

spring:
  datasource:
    url: jdbc:mysql://db:3306/doore?serverTimezone=Asia/Seoul&characterEncoding=UTF-8
    username: # 시크릿 처리된 값
    password: # 시크릿 처리된 값
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

## 해결 방법

해결 방법은 ">>"을 ">"로 수정해 덮어씌우도록 변경하면 됩니다.

~~~yml
- name: Create application.yml
  working-directory: ./src/main/resources
  env:
      APPLICATION_YML: \$ {{ secrets.APPLICATION_YML }}
  run: | # Here!
      echo '\$ {{ secrets.APPLICATION_YML }}' > application.yml 
~~~

## 이전 코드는 왜 ">>" 이였나?

과거 톰 선배님이 작성한 코드는 ">>"을 사용한 코드였습니다. 이를 이해하기 위해서는 Doo-re 프로젝트와 KEEPER 프로젝트의 application.yml 파일(이하 yml 파일) 구조 차이를 알아야 합니다. Doo-re 프로젝트에서는 민감한 정보가 함께 들어간 yml 파일을 깃 이그노어 처리를 하지 않고 있고, KEEPER 프로젝트에서는 민감한 정보가 들어간 yml 파일과 공개해도 되는 yml 파일을 분리하고 있습니다.

현재 민감 정보가 섞인 Doo-re의 yml 파일을 가져와서 분리한다면 아래의 모습이 됩니다.

~~~yml
# 민감하지 않은 정보가 담긴 yml (깃 이그노어 처리를 안 해도 됨)
spring:
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

~~~yml
# 민감한 정보가 담긴 yml (깃 이그노어 처리를 해야 함)
spring:
  datasource:
    url: jdbc:mysql://localhost:3306/doore?serverTimezone=Asia/Seoul&characterEncoding=UTF-8
    username: root
    password: 1234
    driver-class-name: com.mysql.cj.jdbc.Driver
~~~

이렇게 분리한다면 아래와 같은 장점을 얻을 수 있습니다.

- CI/CD 파이프라인 프로세스 단순화

CI/CD 파이프라인에서 민감한 데이터를 위한 별도의 파일이 있으면 배포 중 애플리케이션에 ">>" 처럼 라인을 주입시키는 형식으로 단순화 할 수 있습니다. 만약 하나의 yml에 시크릿 값들을 모아두고 IDE에서 제공하는 설정에 의존한다면 변수를 파이프라인에 삽입하려는 추가적인 단계가 필요하고 하나의 IDE에 의존적이게 됩니다. 개발 도구와는 독립적으로 유지하는 것이 좋다고 생각합니다.

- 분리형 관리

환경 변수가 늘어남에 따라 관리가 어려워질 수 있습니다. 별도의 파일로 분리한다면 보다 명확하고 명시적입니다. 예를 들어, 메일 시스템을 개발할 때는 실제로 발급받은 username과 password를 넣어야 하는데 이는 민감 정보이므로 현재 yml 파일에 담을 수 없습니다. IDE에서 제공하는 기능을 이용하면 위에서 설명한 문제가 발생하게 됩니다.

- 보안

.gitignore 파일을 사용해서 Git과 같은 버전 제어 시스템에서 제외할 수 있습니다. 과거 민감한 데이터를 Push 했던 경험이 있는데 이와 같은 실수를 줄일 수 있습니다.



## 결론

caused by: org.yaml.snakeyaml.constructor.duplicatekeyexception at safeconstructor.java 에러가 무엇이고 해결 방법에 대해서 알아봤습니다. 과거 선배님들의 코드를 보고 왜 동일한 문제가 발생하지 않고 잘 작동되는지를 생각해 봤을 때 민감한 yml파일을 별도로 분리했기 때문입니다. 분리한다면 보안적으로도 우수하고 CI/CD의 프로세스를 굉장히 단순화 시킬 수 있었습니다.