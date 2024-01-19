---
published: true
title:  "It looks like you might be trying to authenticate with OIDC. Did you mean to set the id-token permission? If you are not trying to authenticate with OIDC and the action is working successfully, you can ignore this message."
categories:
  - trouble
---

## 해결 참조 문서

공식 문서를 통해서 해결할 수 있습니다.

["Configure AWS Credentials" Action for GitHub Actions](https://github.com/marketplace/actions/configure-aws-credentials-action-for-github-actions#overview)

## 해결 코드

~~~yml
name: Deploy BDD Development Server

on: workflow_dispatch

jobs:
  deploy_development:
    runs-on: ubuntu-22.04
    timeout-minutes: 10

    # These permissions are needed to interact with GitHub's OIDC Token endpoint.
    # 이거 추가!
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