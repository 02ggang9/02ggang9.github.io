---
published: true
title:  "Infra - Gradle Cache"
categories:
  - infra
---

## 서론

저번 포스팅에서 CICD에 사용되는 시간은 한정된 리소스이기 때문에 Gradle Cache를 사용해 리소스를 절약하는 방법에 대해서 알아보겠다고 했습니다.

아래에서 Github Docs와 톰 선배님의 블로그를 참고해 Gradle을 캐싱하는 방법에 대해서 알아보도록 하겠습니다.

## About caching workflow dependencies

Gradle은 다운받은 의존성을 로컬 캐시에 저장합니다. 하지만 GitHub-hosted runner(이하 러너)는 이전 포스팅 [Infra - About GitHub-hosted runners](https://02ggang9.github.io/infra/GithubAction/)에서 알아봤듯이 Job이 끝나면 프로비저닝된 VM은 폐기된다고 했습니다. 이 말은 러너는 항상 clean한 이미지에서 시작하기 때문에 매번 디펜던시를 다운받게 됩니다. 아래는 Gradle Cache가 적용되기 전 두레 CICD 과정입니다.

![gradle1](https://github.com/02ggang9/02ggang9.github.io/blob/master/_posts/images/infra/githubAction/gradle1.png?raw=true)

깃허브에서는 이런 문제점을 해결하고자 Cache Action을 제공합니다. 아래의 사진에서 확인할 수 있는 패키지 매니저를 사용한다면 최소한의 디펜던시 구성요소를 제공해 주고 디펜던시 캐시를 저장할 수 있도록 해줍니다. 두레는 Gradle을 사용하기 때문에 setup-java action을 사용하도록 하겠습니다.

![gradle1](https://github.com/02ggang9/02ggang9.github.io/blob/master/_posts/images/infra/githubAction/gradle2.png?raw=true)

## Using the cache action

actions/cache를 사용하는 방법은 아래와 같습니다.

~~~yml
  - name: Cache Gradle
    id: cache-gradle
    uses: actions/cache@v3
    with:
      # 캐싱할 경로를 뜻합니다. <Required>
      path: ~/.gradle/caches

      # 캐시를 저장하기 위한 키(512자가 넘으면 액션 실패) <Required>
      # 러너의 운영체제별로 구분하고 캐시의 범주를 나타내는 gradle, .gradle 파일을 해시화
      key: ${{ runner.os }}-gradle-${{ hashFiles('**/*.gradle') }}

      # 캐싱 히트에 실패할 경우 가장 가까운 캐시를 찾기 위함 <Optional>
      restore-keys: ${{ runner.os }}-gradle-
~~~

위의 코드를 그대로 사용하면 되겠다고 생각했지만 톰 선배님의 블로그에서 아래와 같은 내용을 봤습니다.

> gradle 캐싱 적용을 위해 찾아본 많은 글들이 위 코드와 같이 key 값을 hashFiles 함수를 이용했습니다. hashFiles이란 이름에서 알 수 있듯이 file의 hash 값을 반환합니다. 파일 내용이 변경이 된다면 hash 값도 바뀔테니 cache miss가 일어날 겁니다. 하지만 이 점에 대한 의문이 있습니다. 보통 Spring Boot .gitignore 내용을 보면 .gradle가 추가 되어있습니다. GitHub-hosted runner 환경이라면 매번 새로운 환경에서 빌드가 될텐데 어떻게 .gradle 파일이 변경 되었는지 확인 할 수 있을까요. .gitignore에 .gradle이 추가되어있으면 사실 저 위 코드는 의미가 없어지는것이 아닌가 생각하였습니다.

두레 프로젝트는 Spring Boot를 사용하기 때문에 위의 내용과 동일하게 .gitignore에 .gradle이 들어가 있습니다. 하지만 다른 분들의 CICD 로그를 봤을 때 이상하게도 캐싱이 잘 됩니다. 

## Using the Gradle Build Action

위와 같은 의문을 해결하기 위해 action/cache의 issues를 찾아보던 중 [Update documentation to recommend gradle/gradle-build-action for caching Gradle builds](https://github.com/actions/cache/issues/1247)을 발견했고 이 글의 내용은 gradle-build-action을 사용하라는 내용이였습니다.

### gradle-build-action

[Execute Gradle builds in GitHub Actions workflows](https://github.com/marketplace/actions/gradle-build-action) 공식 문서에 적힌 이점과 사용 방법은 아래와 같습니다.

- gradle 버전을 명시해 "자동으로" Gradle distribution을 다운하고 캐싱

- setup-java와 actions/cache를 사용하는 것보다 훨씬 효과적인 캐싱을 제공

- 상세한 캐싱 사용 결과를 제공

~~~yml
- name: Gradle Build & Test
  uses: gradle/gradle-build-action@v2
  with:
      gradle-version: 8.5
      arguments: build # ./gradlew build
      cache-read-only: ${{ github.ref != 'refs/heads/main' && github.ref != 'ref/heads/develop' }} # Github-flow 전략에 따라 main과 develop 브랜치만 캐쉬를 write할 수 있도록 함
~~~

의문이 가득하고 복잡한 key option 설정없이 gradle-version만 명시하면 훨씬 우수한 성능으로 자동 캐싱해준다는 점이 마음에 들어 두레 CICD에 적용했습니다. 

## 적용 후 결과

|Step|적용 전|적용 후|
|----------|---------|--------|
|Total|2m 31s|1m 40s|
|Gradle Build|47s|26s|

![gradle1](https://github.com/02ggang9/02ggang9.github.io/blob/master/_posts/images/infra/githubAction/gradle3.png?raw=true)


## 결론

GitHub-hosted runner는 매번 clean한 이미지에서 시작하기 때문에 항상 Gradle 디펜던시를 다운받는 문제점이 있었습니다. 그 문제점을 해결하기 위해 공식 문서는 [actions/cache](https://github.com/marketplace/actions/cache)를 사용하라고 안내하지만 [actions/gradle-build-action](https://github.com/marketplace/actions/gradle-build-action)을 사용하는 것이 성능적으로 우수하기 때문에 두레 CICD는 actions/gradle-build-cation을 선택했습니다.

성능은 1분 정도 감소하였고 이는 엄청난 발전이라고 생각합니다. 다음은 Docker Cache에 대해서 알아보도록 하겠습니다.