---
published: false
title:  "Infra - Doo-re Infra(6) - Git Action을 위한 권한 설정 및 트러블 슈팅"
categories:
  - infra
---

## Configure AWS Credentials

### 자격 증명 공급자

![ec2](https://github.com/02ggang9/02ggang9.github.io/blob/master/_posts/images/infra/infra6/credentials1.png?raw=true)

![ec2](https://github.com/02ggang9/02ggang9.github.io/blob/master/_posts/images/infra/infra6/credentials2.png?raw=true)

![ec2](https://github.com/02ggang9/02ggang9.github.io/blob/master/_posts/images/infra/infra6/role1.png?raw=true)

![ec2](https://github.com/02ggang9/02ggang9.github.io/blob/master/_posts/images/infra/infra6/role2.png?raw=true)

![ec2](https://github.com/02ggang9/02ggang9.github.io/blob/master/_posts/images/infra/infra6/role3.png?raw=true)

![ec2](https://github.com/02ggang9/02ggang9.github.io/blob/master/_posts/images/infra/infra6/role4.png?raw=true)




~~~yml
name: Deploy BDD Development Server

on: workflow_dispatch

jobs:
  deploy_development:
    runs-on: ubuntu-22.04
    timeout-minutes: 10

    # These permissions are needed to interact with GitHub's OIDC Token endpoint.
    permissions:
      id-token: write
      contents: read

    steps:
      - name: Checkout
        uses: actions/checkout@v4

      - name: Configure AWS Credentials
        uses: aws-actions/configure-aws-credentials@v4
        with:
          role-to-assume: ${{ secrets.AWS_ROLE_ARN }}
          aws-region: ${{ secrets.AWS_REGION }}
~~~

GITHUB ACTION에서 AWS API를 호출하기 위한 권한 설정을 하겠습니다. 

