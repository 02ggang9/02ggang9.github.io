---
published: true
title:  "Infra - Doo-re Infra(6) - ECR 사용을 위한 환경 설정"
categories:
  - infra
---

## 서론

BDD(부산 개발자 소모임)의 Doo-re 프로젝트에서 인프라를 담당하게 되었습니다. BDD는 문서화를 굉장히 중요시하기 때문에 인프라를 구축하면서 필요한 개념과 과정들을 모두 작성하기로 했습니다.

이번 문서에서는 ECR 사용을 위한 환경 설정에 대해서 알아보도록 하겠습니다.

## ECR을 위한 AWS CLI 설정

![ec2](https://github.com/02ggang9/02ggang9.github.io/blob/master/_posts/images/infra/infra7/ecr3.png?raw=true)

![ec2](https://github.com/02ggang9/02ggang9.github.io/blob/master/_posts/images/infra/infra7/ecr4.png?raw=true)

![ec2](https://github.com/02ggang9/02ggang9.github.io/blob/master/_posts/images/infra/infra7/ecr5.png?raw=true)

아래의 커맨드를 EC2 인스턴스에서 그대로 작성합니다.

~~~sh

$ aws configure
AWS Access Key ID [None] : [발급받은 IAM의 Access Key ID]
AWS Secret Access Key [None] : [발급받은 IAM의 Secret Access Key]
Default region name [None] : ap-northeast-2[서울 리전]
Default output format [None] : text / json / table

~~~

## ECR Repository 생성

![ec2](https://github.com/02ggang9/02ggang9.github.io/blob/master/_posts/images/infra/infra7/ecr1.png?raw=true)

![ec2](https://github.com/02ggang9/02ggang9.github.io/blob/master/_posts/images/infra/infra7/ecr2.png?raw=true)


## ECR에 Docker image push

생성된 ECR Repo에서 푸쉬 명령 보기를 누른 후 가이드 해주는 커맨드를 그대로 입력하시면 도커 이미지가 성공적으로 올라갑니다.

![ec2](https://github.com/02ggang9/02ggang9.github.io/blob/master/_posts/images/infra/infra7/ecr6.png?raw=true)

![ec2](https://github.com/02ggang9/02ggang9.github.io/blob/master/_posts/images/infra/infra7/ecr7.png?raw=true)

![ec2](https://github.com/02ggang9/02ggang9.github.io/blob/master/_posts/images/infra/infra7/ecr8.png?raw=true)

