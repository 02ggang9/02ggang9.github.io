---
published: true
title:  "docker :: invalid reference format: repository name must be 에러 해결"
categories:
  - trouble
---

[docker :: invalid reference format: repository name must be](https://toramko.tistory.com/entry/docker-invalid-reference-format-repository-name-must-be-lowercase-%EC%97%90%EB%9F%AC-%ED%95%B4%EA%B2%B0)

~~~sh
### Deploy
docker-compose -p doo-re up -d

### Cleanup
docker rmi $(sudo docker images -f "dangling=true" -q)
~~~

위의 자동화 배포 스크립트를 아래로 수정했습니다. 이 문제는 docker 문법이 올바르지 않아 발생한 문제입니다.

~~~sh
### Deploy
docker-compose up -d

### Cleanup
docker rmi $(docker images --filter "dangling=true" -q)
~~~