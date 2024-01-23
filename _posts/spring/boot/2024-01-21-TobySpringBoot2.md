---
published: true
title:  "Spring Boot - 스프링 부트란 무엇인가?"
categories:
  - spring
---

## 

구성 정보를 담고 있는 Configuration Metadata랑 POJO를 조합해서 사용 가능한 완전히 구성된 시스템을 완성하는거임
스프링 컨테이너 안에 내부에서 우리가 만든 비지니스 오브젝트와 메타데이터를 이용해서 빈을 구성함. 그리고 서버 애플리케이션으로 만들어줌.

##

딱 한번만 만듬. 다른 오브젝트한테 기능을 수행해달라고 요청할 수 있음. controller -> service 책임이 훨씬 줄어듬.

Controller의 역할 중 하나는 요청의 검증임. 그래서 파라미터로 name이 오길 기대했지만 아니라면 작업을 수행할 수 없다고 에러 메시지를 보내줘야 함.