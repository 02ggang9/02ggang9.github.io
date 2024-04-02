---
published: false
title:  "Kubernetes - ReplicaSet과 Deployment"
categories:
  - infra
---

## 서론

[Kubernetes](https://kubernetes.io/ko/docs/concepts/workloads/controllers/deployment/) 공식 문서와 [대세는 쿠버네티스](https://www.inflearn.com/course/lecture?courseSlug=%EC%BF%A0%EB%B2%84%EB%84%A4%ED%8B%B0%EC%8A%A4-%EA%B8%B0%EC%B4%88&unitId=23473&tab=curriculum) 강의를 보고 ReplicaSet과 Deployment에 대해서 정리했습니다.

## ReplicaSet

### 정의

> 레플리카셋의 목적은 파드 집합의 실행을 항상 안정적으로 유지하는 것이다.

다들 레플리카셋이 파드의 수를 유지시켜 주는 역할로 알고 있고, 맞습니다. 하지만 레플리카셋만 사용하는 경우가 있는데 이는 쿠버네티스에서 권장하고 있지 않습니다. 아래에서 볼 Deployment를 사용해서 ReplicaSet을 관리하라고 권장하고 있습니다.

### 간단한 실습

~~~yml
apiVersion: apps/v1
kind: ReplicaSet
metadata:
  name: redis-rs
spec:
  replicas: 2
  selector:
    matchLabels:
      app: echo
      tier: app
  template:
    metadata:
      labels:
        app: echo
        tier: app
    spec:
      containers:
        - name: redis
          image: redis:7-alpine3.17
~~~

![controller1](https://github.com/02ggang9/02ggang9.github.io/blob/master/_posts/images/infra/controller1/controller1.png?raw=true)

위의 사진을 보면 pod/redis-rs-[hash] 형태의 2개의 파드가 뜬 것을 확인하실 수 있습니다. 두 파드의 label의 키 값은 app, tier가 있는데 파드 중 한 개의 라벨을 때버리면 레플리카 셋의 가용성 유지 때문에 새로운 파드가 하나 더 뜨는 것을 확인하실 수 있습니다. (redis-rs-2gjjs의 태그가 하나 없어지고, redis-rs-k7fh9에 새로운 파드가 생김)

![controller2](https://github.com/02ggang9/02ggang9.github.io/blob/master/_posts/images/infra/controller1/controller2.png?raw=true)

위의 yml 파일을 보면 matchLabel과 template.metadata.labels의 라벨이 동일한데, 서로 다르게 정의하면 selector의 matchLabel과 일치하지 않기 때문에 계속해서 파드를 생성하려고 합니다. 따라서 이런 리소스 낭비를 막기 위해서 라벨이 일치하지 않으면 api 호출 단에서 막아버리는 모습을 확인하실 수 있습니다.

## Deployment

### 정의
> 디플로이먼트는 파드와 레플리카셋에 대한 선언적 업데이트를 제공한다. 

레플리카셋의 상위 개념이고, 레플리카 셋과 완전히 다른게 아니라, ReplicaSet + 버전 관리 기능이 추가되었다고 생각하시면 쉽습니다. 

### 간단한 실습(Deployment로 Pod 생성)

~~~yml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: nginx-deployment
  labels:
    app: nginx
spec:
  replicas: 3
  selector:
    matchLabels:
      app: nginx
  template:
    metadata:
      labels:
        app: nginx
    spec:
      containers:
      - name: nginx
        image: nginx:1.14.2
        ports:
        - containerPort: 80
~~~

![controller3](https://github.com/02ggang9/02ggang9.github.io/blob/master/_posts/images/infra/controller1/controller3.png?raw=true)

위의 결과를 보시면 총 3개의 파드가 생성된 것을 확인하실 수 있습니다. 그 뿐만 아니라 deployment와 relicaSet도 함께 생성되었는데 이름에 해쉬 값이 들어있는 것을 확인하실 수 있습니다.

쿠버네티스를 처음 공부할 때 Pod -> ReplicaSet -> Deployment 순서로 공부하는데 파드가 만들어지는 순서는 Deployment -> ReplicaSet -> Pod 순으로 만들어 집니다. 처음 Deployment가 생성되고 ReplicaSet이 만들어질 때, [DeploymentName]-[무작위 Hash]의 이름으로 만들어지고, ReplicaSet이 Pod를 만들 때, [DeploymentName]-[무작위 Hash]-[무작위 Hash]로 이름이 만들어 집니다.

### 간단한 실습(Deployment Update)

~~~yml
kubectl set image deployment/nginx-deployment nginx=nginx:1.16.1
kubectl get all 
~~~

![controller4](https://github.com/02ggang9/02ggang9.github.io/blob/master/_posts/images/infra/controller1/controller4.png?raw=true)

nginx의 태그를 수정하면 deployment가 변경을 감지하고, 새로운 replicaSet을 만들고, 새로운 pod또한 만드는 모습을 확인하실 수 있습니다.

DeploymentController는 각 주기마다 새로운 디플로이먼트에서 레플리카 셋이 의도한 파드를 생성하고 띄우는 것을 주시합니다. 이를 롤 오버라고 하는데, 간단한 예시를 보면 쉽게 이해하실 수 있습니다.

> 예를 들어 디플로이먼트로 nginx:1.14.2 레플리카를 5개 생성을 한다. 하지만 nginx:1.14.2 레플리카 3개가 생성되었을 때 디플로이먼트를 업데이트해서 nginx:1.16.1 레플리카 5개를 생성성하도록 업데이트를 한다고 가정한다. 이 경우 디플로이먼트는 즉시 생성된 3개의 nginx:1.14.2 파드 3개를 죽이기 시작하고 nginx:1.16.1 파드를 생성하기 시작한다. 이것은 과정이 변경되기 전 nginx:1.14.2 레플리카 5개가 생성되는 것을 기다리지 않는다.

### 간단한 실습(Deployment rollback)

~~~yml
kubectl set image deployment/nginx-deployment nginx=nginx:1.161
kubectl rollout undo deployment/nginx-deployment --to-revision=2
~~~

![controller5](https://github.com/02ggang9/02ggang9.github.io/blob/master/_posts/images/infra/controller1/controller5.png?raw=true)

위의 스크립트로 일부러 없는 이미지를 띄워 이미지 풀 루프에 고착된 상태로 만들고, 1.16.1 버전으로 롤백시키는 모습입니다. 레플리카 셋을 활용하면 이런 버전 관리가 어려울텐데, Deployment로 파드를 배포한다면 버전 관리가 굉장히 쉬운 것을 체감할 수 있습니다.

### 기타 설정(MaxUnavailable, MaxSurge, progressDeadlineSeconds)

MaxUnavailable은 업데이트 중 사용할 수 없는 최대 파드의 수를 지정하는 필드입니다. 절대 숫자(EX 1) 또는 퍼센트로 설정할 수 있는데 1로 설정하게 되면 파드 1개만 이전 버전으로 남기고 다른 파드를 스케일 다운할 수 있습니다. 다른 파드의 스케일 다운이 완료되면 나머지 1개의 파드도 스케일 다운을 하게 됩니다.

MaxSurge는 생성할 수 있는 최대 파드의 수를 지정하는 필드입니다. 최대 파드 수를 정하는 이유는 스케일 다운할 때 기존 파드를 두고, 새로운 파드를 미리 만들어 두는 과정이 있는데, 이때 생성 리소스가 너무 커져 노드의 리소스가 바닥나는 일을 방지하기 위해서입니다.

ProgressDeadlineSeconds는 디플로이먼트가 진행되는 것을 대기시키는 시간을 명시하는 필드입니다. 기본 값이 600초인데, 배포하는 시간이 600초를 넘어가면 그 즉시 디플로이먼트를 롤백한다고 합니다.

## 결론

레플리카 셋은 파드의 가용성을 유지하기 위해 노력합니다. 하지만 레플리카 셋을 단독으로 사용하는 것은 쿠버네티스 측에서 권장하지 않습니다. 버전 관리에 용이한 디플로이먼트를 사용해서 레플리카 셋을 관리하고, 파드를 관리하는 것을 권장합니다. 디플로이먼트에서 배포하는 과정과 롤백하는 과정은 매우 간단했습니다. 

## 의문점

실무자 분들은 롤백을 어떻게 시키는지 궁금합니다. 다들 쿠버네티스의 기본은 아셔서 history를 뒤져보고 롤백 하시는 건지, 아니면 자동화된 스크립트를 만들고 사용하는지 개인적으로 궁금했습니다.

또, 최근 초보(?)님의 글을 보고 든 생각인데, 쿠버네티스의 스케일 업 기능을 사용할 때, 서버 스펙 또한 수직적으로 확장시켜야 하는게 아닌지? 생각이 들었습니다. 유튜브에 배민 레퍼런스 같은 것을 보면 트래픽이 많을 때는 스케일링을 통해 안정적으로 트래픽을 받을 수 있었다고 뭉뚱그려서 설명하시는데, 어떤식으로 스케일링을 통하는지 궁금하네요..

빨리 센디 같은 회사에 인턴으로 참여해서 이런 경험을 해보고 싶네요..ㅎㅎ

