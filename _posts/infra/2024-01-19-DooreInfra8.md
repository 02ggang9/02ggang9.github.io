---
published: true
title:  "Infra - Doo-re Infra(6) - ECR 사용을 위한 환경 설정"
categories:
  - infra
---

## ECR을 위한 AWS CLI 설정

![ec2](https://github.com/02ggang9/02ggang9.github.io/blob/master/_posts/images/infra/infra7/ecr3.png?raw=true)

![ec2](https://github.com/02ggang9/02ggang9.github.io/blob/master/_posts/images/infra/infra7/ecr4.png?raw=true)

![ec2](https://github.com/02ggang9/02ggang9.github.io/blob/master/_posts/images/infra/infra7/ecr5.png?raw=true)

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

![ec2](https://github.com/02ggang9/02ggang9.github.io/blob/master/_posts/images/infra/infra7/ecr6.png?raw=true)

![ec2](https://github.com/02ggang9/02ggang9.github.io/blob/master/_posts/images/infra/infra7/ecr7.png?raw=true)

