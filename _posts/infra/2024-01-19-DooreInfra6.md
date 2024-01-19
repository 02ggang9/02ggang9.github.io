---
published: true
title:  "Infra - Doo-re Infra(5) - EC2 환경 설정과 도커 띄우기"
categories:
  - infra
---

## 서론

BDD(부산 개발자 소모임)의 Doo-re 프로젝트에서 인프라를 담당하게 되었습니다. BDD는 문서화를 굉장히 중요시하기 때문에 인프라를 구축하면서 필요한 개념과 과정들을 모두 작성하기로 했습니다.

이번 문서에서는 EC2 환경 설정 과정과 컨테이너를 실행시킨 후 성공적으로 접속이 잘 되는지 확인하는 과정을 알아보겠습니다.

## AWS EC2 접속

EC2 인스턴스에 접근하는 방법은 크게 EC2 Instance Connect를 사용한 방법과 SSH 클라이언트를 사용한 방법이 있습니다. EC2 Instance Connect는 AWS 웹 사이트에서 인스턴스에 접속하는 방법이라 굉장히 불편합니다. SSH 클라이언트는 터미널 창을 이용할 수 있어 익숙하지만 아래처럼 key 이름과 사용자 이름, 퍼블릭 IP 주소를 입력해야 하는 과정이 매우 번거롭습니다.

~~~sh
ssh -i "your-key-pair.pem" ubuntu@ec2-00-000-000-00.ap-northeast-2.compute.amazonaws.com
~~~

이 번거로운 과정을 피하기 위해서 과거 톰 선배님께서는 아래의 사이트 방식을 추천해 주셨고 저 또한 매우 즐겨 사용하고 있습니다.

[Mac에서 AWS EC2 접속 빠르게 하기](https://vanillacreamdonut.tistory.com/313)


## Swap Memory 설정

스왑 메모리란 RAM이 부족할 경우 HDD의 일정 공간을 RAM 처럼 사용하는 것입니다. 처음 인스턴스를 만들고 [Doo-re Infra(3) - Infra Code 작성 및 로컬 환경에서 테스트](https://02ggang9.github.io/infra/DooreInfra4/)에서 만든 도커 컨테이너 2개를 띄웠는데 인스턴스가 뻗어버리는 현상이 발생했습니다. 인스턴스와의 연결도 불가능해 인스턴스를 중지했다가 재시작하는 과정을 4번정도 반복했는데 스왑 메모리 설정을 통해 문제를 해결했습니다.

스왑 메모리를 설정하는 자세한 방법은 아래의 두 사이트에서 확인하실 수 있습니다.

[스왑 파일을 사용하여 Amazon EC2 인스턴스에서 스왑 공간으로 사용할 메모리를 할당하는 방법은 무엇입니까?](https://repost.aws/ko/knowledge-center/ec2-memory-swap-file)

[AWS EC2 프리티어에서 메모리 부족현상 해결방법](https://sundries-in-myidea.tistory.com/102)


## 도커 설치 및 컨테이너 실행

### 도커 설치 및 도커 컴포즈 설치

도커를 설치하는 방법은 아래의 블로그에서 확인하실 수 있습니다.

[[Docker] Ubuntu 22.04 Docker 설치](https://velog.io/@osk3856/Docker-Ubuntu-22.04-Docker-Installation)


### 도커 이미지 빌드

~~~sh

git clone https://github.com/02ggang9/01-doo-re-infrastructure.git
cd 01-doo-re-infrastructure/doo-re/build/

sudo docker build -t doo-re-app:build -f docker/app.dockerfile .
sudo docker build -t doo-re-db:build -f docker/db.dockerfile .

~~~

### .env 파일 생성

~~~sh
MYSQL_DATABASE=
MYSQL_USER=
MYSQL_PASSWORD=
MYSQL_ROOT_PASSWORD=
~~~

### docker-compose 실행

~~~sh
sudo docker-compose up
~~~

## 결과

아래의 사진처럼 정상적으로 빌드 후 Spring Boot가 실행되고 EC2 퍼블릭 IP 주소로 접근할 때 White Label Error 페이지가 뜬다면 성공입니다.

![ec2](https://github.com/02ggang9/02ggang9.github.io/blob/master/_posts/images/infra/infra5/result1.png?raw=true)

![ec2](https://github.com/02ggang9/02ggang9.github.io/blob/master/_posts/images/infra/infra5/result2.png?raw=true)



## 결론

SSH alias로 쉽게 EC2 인스턴스에 접근하는 방법에 대해서 알아봤습니다. 또, 도커 컨테이너를 실행했을 때 인스턴스가 뻗는 문제를 해결하기 위해서 스왑 메모리를 설정했습니다. 그 이후에는 env 파일 설정 후 로컬에서 작성한 코드 그대로 가져와서 이미지 빌드 후 실행시켜보는 과정을 거쳤습니다. 

이제 수동 배포를 하는 과정을 이해할 수 있습니다. EC2 인스턴스에 접속한 후 도커 이미지를 새로 빌드에 docker-compose를 이용해서 컨테이너를 실행하기만 하면 됩니다. 하지만 dev 환경에서는 코드 변경이 자주 일어나기 때문에 수동 배포는 정말 귀찮습니다. 다음 포스팅부터는 자동 배포를 위한 과정에 대해서 알아보도록 하겠습니다.

