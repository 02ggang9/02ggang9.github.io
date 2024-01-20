---
published: true
title:  "Infra - Doo-re Infra(7) - S3 버킷 생성 방법과 Slack 연동 방법"
categories:
  - infra
---

## 서론



## 본론

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

![ec2](https://github.com/02ggang9/02ggang9.github.io/blob/master/_posts/images/infra/infra8/slack1.png?raw=true)

![ec2](https://github.com/02ggang9/02ggang9.github.io/blob/master/_posts/images/infra/infra8/slack2.png?raw=true)

![ec2](https://github.com/02ggang9/02ggang9.github.io/blob/master/_posts/images/infra/infra8/slack3.png?raw=true)

![ec2](https://github.com/02ggang9/02ggang9.github.io/blob/master/_posts/images/infra/infra8/slack4.png?raw=true)

![ec2](https://github.com/02ggang9/02ggang9.github.io/blob/master/_posts/images/infra/infra8/slack5.png?raw=true)



## 결론


