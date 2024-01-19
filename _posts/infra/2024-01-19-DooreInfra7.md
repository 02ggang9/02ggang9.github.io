---
published: true
title:  "Infra - Doo-re Infra(6) - Git Action을 위한 권한 설정 Configure AWS Credntials"
categories:
  - infra
---

## 서론

BDD(부산 개발자 소모임)의 Doo-re 프로젝트에서 인프라를 담당하게 되었습니다. BDD는 문서화를 굉장히 중요시하기 때문에 인프라를 구축하면서 필요한 개념과 과정들을 모두 작성하기로 했습니다.

이번 문서에서는 Git Action을 위한 Configure AWS Credntials 설정 방법을 알아보도록 하겠습니다.

## 자격 증명 공급자 설정

![ec2](https://github.com/02ggang9/02ggang9.github.io/blob/master/_posts/images/infra/infra6/credentials1.png?raw=true)

공급자 URL과 대상은 아래의 코드를 복사 붙여넣기 하시면 됩니다. 아래의 블로그들을 참고했습니다.

[Amazon Web Services에서 OpenID Connect 구성](https://docs.github.com/ko/enterprise-cloud@latest/actions/deployment/security-hardening-your-deployments/configuring-openid-connect-in-amazon-web-services)

[[CI/CD] GitHub Action AWS에 IAM Role로 접근하기](https://zerone-code.tistory.com/11)

~~~sh
공급자 URL: https://token.actions.githubusercontent.com
대상: sts.amazonaws.com
~~~

![ec2](https://github.com/02ggang9/02ggang9.github.io/blob/master/_posts/images/infra/infra6/credentials2.png?raw=true)

## Role 설정

![ec2](https://github.com/02ggang9/02ggang9.github.io/blob/master/_posts/images/infra/infra6/role1.png?raw=true)

![ec2](https://github.com/02ggang9/02ggang9.github.io/blob/master/_posts/images/infra/infra6/role2.png?raw=true)

GITHUB ACTION에서 S3에 파일을 업로드하고 EC2에 SSH 접근을 하기 때문에 S3와 EC2 FullAceess 권한을 추가했습니다. (아직 학습이 부족해 FullAcess 권한을 부여했지만 최소한으로 권한을 부여하려면 다른 권한을 부여해주세요)

![ec2](https://github.com/02ggang9/02ggang9.github.io/blob/master/_posts/images/infra/infra6/role3.png?raw=true)

아래의 신뢰 정책에서 repo:BDD-CLUB으로 되어있는데 이는 BDD-CLUB organization에 해당하는 모든 repository에 접근한다는 의미입니다. repository를 특정하고 싶으면 아래와 같이 작성하시면 됩니다.

~~~json
{
    "Version": "2012-10-17",
    "Statement": [
        {
            "Effect": "Allow",
            "Principal": {
                "Federated": "arn:aws:iam::123456789100:oidc-provider/token.actions.githubusercontent.com"
            },
            "Action": "sts:AssumeRoleWithWebIdentity",
            "Condition": {
                "StringEquals": {
                    "token.actions.githubusercontent.com:aud": "sts.amazonaws.com"
                },
                "StringLike": {
                    "token.actions.githubusercontent.com:sub": "repo:02ggang99/01-doo-re-back-action-practice:*" // Here!
                }
            }
        }
    ]
}
~~~

![ec2](https://github.com/02ggang9/02ggang9.github.io/blob/master/_posts/images/infra/infra6/role4.png?raw=true)

## git action 코드 작성

아래의 시크릿 값은 할당 받은 자격 증명 공급자의 ARN을 입력하시면 됩니다.

![ec2](https://github.com/02ggang9/02ggang9.github.io/blob/master/_posts/images/infra/infra6/role5.png?raw=true)

~~~yml
name: Deploy BDD Development Server

on: workflow_dispatch

jobs:
  deploy_development:
    runs-on: ubuntu-22.04
    timeout-minutes: 10

    # These permissions are needed to interact with GitHub's OIDC Token endpoint.
    # 과거 코드에서는 없어도 됐지만 업데이트 후 필수적으로 넣어줘야 합니다.
    permissions:
      id-token: write
      contents: read

    steps:
      - name: Checkout
        uses: actions/checkout@v4

      - name: Configure AWS Credentials
        uses: aws-actions/configure-aws-credentials@v4
        with:
          role-to-assume:  # ${{ secrets.AWS_ROLE_ARN }} 할당 받은 자격 증명 공급자의 ARN
          aws-region: # ${{ secrets.AWS_REGION }} 리전을 설정해주세요.
~~~

## 발생한 트러블과 해결 방법

