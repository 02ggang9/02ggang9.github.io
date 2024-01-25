---
published: true
title:  "Infra - About GitHub-hosted runners"
categories:
  - infra
---

## 서론

두레 프로젝트에서는 CICD 파이프라인을 GitHub Action을 사용해서 구축했습니다. GitHub에서 제공해 주는 runner에서 GitHub action의 workflow를 실행시키기 때문에 기본적인 지식은 알고 있어야 한다고 생각했습니다.

아래의 내용은 [About GitHub-hosted runners](https://docs.github.com/en/actions/using-github-hosted-runners/about-github-hosted-runners/about-github-hosted-runners#overview-of-github-hosted-runners) 공식 문서를 참고했고 두레 인프라를 구축하면서 필요했던 내용을 정리했습니다. 보다 자세한 내용은 공식 문서에서 참고하실 수 있습니다.


## Using a GitHub-hosted Runner

GitHub-hosted runner(이하 러너)는 깃허브 액션 workflow를 실행하기 위한 머신입니다. 깃허브에서 macOS, Ubuntu Linux, Windows 세 가지의 OS를 프로비저닝(provisioning)합니다. Job이 사작하면 자동적으로 새로운 VM을 프로비저닝하게 됩니다. 모든 Step은 이 VM에서 동작하고 러너의 FileSystem을 공유하고 Docker Container를 실행시킬 수도 있습니다. Job이 끝나면 자동적으로 VM은 폐기됩니다. 아래의 사진처럼 두 개의 Job이 서로 다른 VM에서 돌아갑니다.

![vm1](https://github.com/02ggang9/02ggang9.github.io/blob/master/_posts/images/infra/githubAction/vm1.png?raw=true)

## Standard GitHub-hosted runners for Public repositories

깃허브에서 제공하는 러너의 사양은 굉장히 좋습니다. 또, 한달 최소 2,000분 정도의 넉넉한 시간을 제공하기 때문에 Doo-re 프로젝트에서 시간 초과가 날 가능성은 없을 것 같습니다.

![vm1](https://github.com/02ggang9/02ggang9.github.io/blob/master/_posts/images/infra/githubAction/vm2.png?raw=true)

![vm1](https://github.com/02ggang9/02ggang9.github.io/blob/master/_posts/images/infra/githubAction/vm3.png?raw=true)

Private Repository는 사양이 반토막 난 러너를 제공하고 비용까지 지불해야 합니다. 또, Windows는 2 Minute multiplier, macOS는 10 Minute multiplier가 적용된다고 하니 주의하시면서 사용하시길 바랍니다. 자세한 사항은 [About biling for GitHub Actions - GitHub Docs](https://docs.github.com/en/billing/managing-billing-for-github-actions/about-billing-for-github-actions#included-storage-and-minutes)에서 확인하실 수 있습니다.

## Workflow Continuity

깃허브 액션은 workflow가 시작될 때, Queue 대기열에 진입하려고 하는데 30분 이내로 진입하지 못하면 자동으로 삭제됩니다. 또한 성공적으로 Queue 대기열에 진입했지만, 러너가 45분 안에 처리되지 않아도 삭제됩니다.

## Administrative privileges

Linix와 macOS의 VM은 passwordless sudo를 사용한다고 합니다. 따라서 파일 권한을 수정하고 실행시키는 불필요한 과정을 안 해도 됩니다. 자세한 정보는 [Sudo Manual](https://www.sudo.ws/docs/man/1.8.27/sudo.man/)을 참조하시면 됩니다.

## The etc/hosts file

러너는 etc/hosts라는 파일과 함께 프로비저닝되는데 이는 암호화 화폐 채굴 및 악성 사이트에 대한 액세스를 차단하는 파일입니다. MiningMadness.com 같은 사이트를 localhost로 rerouting 합니다.

## File Systems

깃허브는 VM의 특별한 경로에서 쉘 커맨드를 실행시킵니다. 파일 경로는 static하지 않기 때문에 아래의 제공하는 환경 변수를 통해서 디렉토리에 접근하라고 합니다.

![vm1](https://github.com/02ggang9/02ggang9.github.io/blob/master/_posts/images/infra/githubAction/vm4.png?raw=true)


## 결론

깃허브에서 제공하는 러너는 크게 3가지의 OS(Linux, Windows, macOS)를 지원합니다. 하나의 Job이 실행될 때 새로운 VM을 프로비저닝하고 Job이 끝나면 알아서 폐기됩니다. 하나의 Job 동안 Step 들은 파일을 공유할 수 있습니다. 이 특징을 사용해서 두레 프로젝트에서도 하나의 Step을 위해 여러 Step을 거치며 파일을 다운받고 공유하고 있습니다.

깃허브에서 무료로 제공하는 시간은 동아리 수준의 프로젝트에서 충분합니다. 하지만 리소스이기 때문에 할 수 있다면 절약해야 합니다. 다음 포스팅은 CICD 고도화 중 Gradle 캐시를 사용해서 리소스를 절약하는 방법에 대해서 알아보겠습니다.