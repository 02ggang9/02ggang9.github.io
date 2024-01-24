---
published: true
title:  "Infra - Doo-re Infra(1) - 도커란 무엇인가?"
categories:
  - infra
---

## 서론

BDD(부산 개발자 소모임)의 Doo-re 프로젝트에서 인프라를 담당하게 되었습니다. BDD는 문서화를 굉장히 중요시하기 때문에 인프라를 구축하면서 필요한 개념과 과정들을 모두 작성하기로 했습니다.

이번 문서에서는 도커가 무엇인지 알아보고 도커를 사용해야 하는 이유에 대해서 알아보도록 하겠습니다.

## 도커란 무엇인가

도커는 **컨테이너 기반**의 **가상화** 플랫폼입니다. 

## 가상화

가상화는 물리적 머신에서 여러 개의 가상화 머신을 만드는 프로세스입니다. 대표적으로 VM ware가 있습니다. 가상화를 이용하는 이유는 컴퓨터 자원을 효율적으로 사용하기 위함입니다.

한 개의 OS위에 하나의 테스크를 수행하도록 하는 것이 안정적이고 관리하기 쉽습니다. 하지만 이럴 경우 아래의 사진처럼 서버의 자원을 최대한으로 사용할 수 없습니다.

![도커란무엇인가1](https://github.com/02ggang9/02ggang9.github.io/blob/master/_posts/images/infra/도커란무엇인가/가상화예시1.png?raw=true)

가상화를 사용한다면 아래의 사진처럼 서버의 자원을 효율적으로 사용할 수 있습니다.

![도커란무엇인가2](https://github.com/02ggang9/02ggang9.github.io/blob/master/_posts/images/infra/도커란무엇인가/가상화예시2.png?raw=true)


## 컨테이너

컨테이너는 격리된 공간에서 프로세스가 동작하는 기술입니다. 기존의 가상화 소프트웨어는 OS를 가상화하여 사용했지만 컨테이너 기반의 가상화는 프로세스만을 가상화하기 때문에 훨씬 가볍고 빠릅니다.

![컨테이너예시1](https://github.com/02ggang9/02ggang9.github.io/blob/master/_posts/images/infra/도커란무엇인가/컨테이너예시1.png?raw=true)

OS를 격리하는 방식과 프로세스를 격리시키는 컨테이너 기술의 성능 비교는 아래의 글에서 확인하실 수 있습니다.

[[ Docker # 03 알아보기] - VM vs Docker 성능 비교](https://artistdata.tistory.com/5)

## 이미지

이미지는 *컨테이너를 실행하기 위한 파일*과 *설정 값들*을 정의해 놓은 파일입니다. 아래는 두레 프로젝트의 app 이미지입니다.

~~~dockerfile
# 컨테이너를 실행하기 위한 파일
FROM gradle:8.5.0-jdk17 

WORKDIR /home/doo-re 

# 컨테이너를 실행하기 위한 파일
RUN git clone https://github.com/BDD-CLUB/01-doo-re-back.git . 

# 설정 값
ENV LANG=ko_KR.UTF-8 \
    LANGUAGE=ko_KR.UTF-8 \
    LC_ALL=ko_KR.UTF-8

# 컨테이너를 실행하기 위한 파일
COPY docker/data/entrypoint.sh /entrypoint.sh
COPY docker/data/application.yml /application.yml

~~~

위의 이미지를 보면 **컨테이너를 실행하기 위한 파일**과 **설정 값** 2개 밖에 없습니다. 컨테이너는 이미지를 실행시킨 상태라고 볼 수 있습니다. 

기존의 VM은 필요없는 파일과 설정 값들이 많았지만 이미지는 컨테이너를 실행하기 위해 필요한 최소한의 파일과 설정을 하기 때문에 훨씬 가볍습니다.


## 도커를 사용해야 하는 또 다른 이유

도커를 사용해야 하는 또 다른 이유는 문서화 입니다. 

과거 KEEPER 동아리에서 진행한 악성코드 분석 스터디에서 환경 세팅을 담당했었는데 총 3일이 걸렸습니다. 환경을 세팅하기 위한 모든 과정을 문서화하고 [배포](https://stump-cress-2f5.notion.site/KEEPER-1-c960dd9361c64096b7fbce30f6e7c7f1?pvs=4)했지만 서로 각기 다른 에러가 터져 스터디 진행이 어려웠습니다. 같은 가상화 소프트웨어와 같은 가상화 OS를 설치했지만 다른 패키지가 설치되고 이유 모를 에러가 발생했습니다.

도커는 환경 설정을 위한 모든 과정을 블로깅하거나 메모장에 적어 유지보수 하지 않고 Dockerfile로 관리할 수 있습니다. 또, 다른 의존성 패키지가 설치될 걱정을 할 필요가 없습니다.

이번 인프라를 구성하면서 과거 톰 선배님이 작성한 Dockerfile을 많이 참고했는데 필요한 모든 과정이 코드로 작성이 되어있기 때문에 이해하기 정말 편했고 코드 수정도 거의 없었습니다. 도커는 성능, 유지보수성이 기존의 방식보다 훨씬 뛰어나기 때문에 현대 시대에서는 꼭 알아야하는 필수 지식인 것 같습니다.


## 결론

도커는 *컨테이너 기반*의 *가상화* 플랫폼입니다. 프로세스만 격리시키는 컨테이너 기술을 사용하기 때문에 기존의 방식보다 성능이 훨씬 뛰어나고 자원을 효율적으로 사용할 수 있습니다. 또, Dockerfile이 곧 문서이고 유지 보수성이 편리하기 때문에 현대 시대에서는 꼭 알아야하는 필수 지식입니다.

