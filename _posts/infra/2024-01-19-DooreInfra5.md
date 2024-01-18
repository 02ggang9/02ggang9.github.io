---
published: true
title:  "Infra - Doo-re Infra(4) - AWS EC2 인스턴스 생성"
categories:
  - infra
---


## 서론

BDD(부산 개발자 소모임)의 Doo-re 프로젝트에서 인프라를 담당하게 되었습니다. BDD는 문서화를 굉장히 중요시하기 때문에 인프라를 구축하면서 필요한 개념과 과정들을 모두 작성하기로 했습니다.

이번 문서에서는 EC2 인스턴스 생성과 탄력적 IP를 발급받아보도록 하겠습니다. 이 과정은 박재성님의 비전공자도 이해할 수 있는 AWS 입문/실전 강의를 참고했습니다.


## 리전(Region) 설정하기

![gradle1](https://github.com/02ggang9/02ggang9.github.io/blob/master/_posts/images/infra/infra4/region.png?raw=true)

AWS 로그인 후 위의 사진처럼 아시아 태평양 (서울)로 선택합니다. 만약 웹 애플리케이션 고객이 해외에 거주하신 분들이라면 서울보다는 미국 리전을 선택하시는게 훨씬 좋습니다.

그리고 주의할 점이 앞으로 EC2 인스턴스를 생성할텐데 생성 후 인스턴스가 보이지 않는다면 꼭 리전을 확인해야 합니다.

## EC2 설정

AWS > EC2 검색 > 인스턴스 > 인스턴스 시작(주황색 버튼)

필수적인 요소는 빨간색 테두리로 표시하겠습니다. 테두리가 없는 곳은 굳이 건들 필요가 없습니다.

~~~yml

# 예시
이름: doo-re-dev-server

운영체제: Ubuntu

키 페어 이름: dev-server-key 
이름 선택 후 새 키 페어 생성 버튼 클릭 후 .pem 키 페어 발급

~~~

![ec2](https://github.com/02ggang9/02ggang9.github.io/blob/master/_posts/images/infra/infra4/ec2-1.png?raw=true)


네트워크 설정 오른쪽에 편집 버튼 클릭

보안 그룹 규칙 추가 후 HTTP 유형에 위치 무관으로 설정합니다. (도메인을 발급받고 HTTPS 적용을 한다면 이후 설정에서 보안 그룹 규칙을 HTTPS 유형을 추가하시면 됩니다)

프리티어 인스턴스는 30GiB와 gp3까지는 무료로 제공합니다.

인스턴스 시작 버튼을 클릭하면 인스턴스가 생성됩니다.

![ec2](https://github.com/02ggang9/02ggang9.github.io/blob/master/_posts/images/infra/infra4/ec2-2.png?raw=true)


## 탄력적 IP 발급

탄력적 IP를 적용하지 않은 EC2 인스턴스는 중지 후 재시작을 하면 퍼블릭 IP 주소가 매번 변경됩니다. 이를 방지하기 위해서 탄력적 IP를 발급받아야 합니다.

네트워크 및 보안 > 탄력적 IP > 탄력적 IP 주소 할당 버튼 클릭 > 할당

![ec2](https://github.com/02ggang9/02ggang9.github.io/blob/master/_posts/images/infra/infra4/탄력1.png?raw=true)

![ec2](https://github.com/02ggang9/02ggang9.github.io/blob/master/_posts/images/infra/infra4/탄력2.png?raw=true)


할당된 탄력적 IP 주소 클릭 후 탄력적 IP 주소 연결 버튼 클릭 > 새로 생성한 인스턴스 선택 > 연결 버튼 클릭

![ec2](https://github.com/02ggang9/02ggang9.github.io/blob/master/_posts/images/infra/infra4/탄력3.png?raw=true)

![ec2](https://github.com/02ggang9/02ggang9.github.io/blob/master/_posts/images/infra/infra4/탄력4.png?raw=true)


탄력적 IP 연결을 성공적으로 마칠 경우 아래의 사진처럼 퍼블릭 IP와 탄력적 IP 주소가 동일하게 바뀌고, 인스턴스를 중지 후 시작을 해도 퍼블릭 IP 주소가 변경되지 않습니다.

![ec2](https://github.com/02ggang9/02ggang9.github.io/blob/master/_posts/images/infra/infra4/탄력5.png?raw=true)


## 비용이 나가지 않게 EC2 인스턴스를 삭제

비용이 나가지 않게 EC2 인스턴스를 삭제하려면 우선 탄력적 IP를 릴리즈하고, 발급 받은 탄력적 주소 IP를 반납해야 합니다. 그 후, 인스턴스를 종료하면 비용 걱정은 안 하셔도 됩니다.


## 결론

EC2 인스턴스 생성 방법에 대해서 알아봤습니다. 한번 EC2를 생성해보고 탄력적 IP까지 받아보는 경험을 해보면 인스턴스 생성 공포에서 벗어날 수 있습니다.

다음에는 EC2 인스턴스 환경 구성과 도커를 활용해서 스프링 부트를 띄워보도록 하겠습니다.
