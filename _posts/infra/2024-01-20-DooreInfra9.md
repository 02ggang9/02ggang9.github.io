---
published: true
title:  "Infra - Doo-re Infra(8) - S3 버킷 생성 방법과 Slack 연동 방법"
categories:
  - infra
---

## 서론

BDD(부산 개발자 소모임)의 Doo-re 프로젝트에서 인프라를 담당하게 되었습니다. BDD는 문서화를 굉장히 중요시하기 때문에 인프라를 구축하면서 필요한 개념과 과정들을 모두 작성하기로 했습니다.

이번 문서에서는 S3 버킷 생성과 Slack 연동 방법에 대해서 알아보도록 하겠습니다.

## S3 버킷 생성

![ec2](https://github.com/02ggang9/02ggang9.github.io/blob/master/_posts/images/infra/infra8/s3-1.png?raw=true)

![ec2](https://github.com/02ggang9/02ggang9.github.io/blob/master/_posts/images/infra/infra8/s3-2.png?raw=true)

~~~json
{
    "Version": "2012-10-17",
    "Statement": [
        {
            "Sid": "Statement1",
            "Effect": "Allow",
            "Principal": "*",
            "Action": "s3:GetObject",
            "Resource": "arn:aws:s3:::{내가 만든 buckey 이름}/*"
        }
    ]
}
~~~

## Slack 연동 방법

[Slack Webhook](https://api.slack.com/messaging/webhooks) 사이트에 접속합니다.

![ec2](https://github.com/02ggang9/02ggang9.github.io/blob/master/_posts/images/infra/infra8/slack1.png?raw=true)

![ec2](https://github.com/02ggang9/02ggang9.github.io/blob/master/_posts/images/infra/infra8/slack2.png?raw=true)

![ec2](https://github.com/02ggang9/02ggang9.github.io/blob/master/_posts/images/infra/infra8/slack3.png?raw=true)

![ec2](https://github.com/02ggang9/02ggang9.github.io/blob/master/_posts/images/infra/infra8/slack4.png?raw=true)

마지막으로 Webhook URL을 GITHUB ACTION을 사용할 때 시크릿 값으로 설정하면 됩니다.

![ec2](https://github.com/02ggang9/02ggang9.github.io/blob/master/_posts/images/infra/infra8/slack5.png?raw=true)


## 결론

GITHUB ACTION을 사용하기 위한 모든 밑작업을 마쳤습니다. 다음 글에서는 GITHUB ACTION 코드를 살펴보도록 하겠습니다.
