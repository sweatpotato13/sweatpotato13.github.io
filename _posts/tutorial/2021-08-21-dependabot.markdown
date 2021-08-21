---
layout: post
title: "Dependabot + Github Action으로 dependencies 최신으로 관리하기"
date: "2021-08-21 23:03:44 +0900"
tags: [tutorial]
---

![ss](https://dependabot.com/static/eb991d2434b1b73d4e71145f50359ada/54311/screenshot.png)

`Dependabot`은 dependencies를 쉽게 최신버전으로 관리할 수 있게 해주는 툴입니다. `Dependabot`이 의존성의 최신여부를 체크하여 Pull Request로 각각의 의존성에 대하여 업데이트 할 수 있도록 합니다.

https://dependabot.com/

이 문서에선 Github Action을 이용하여 Dependabot을 사용하는 법을 간단히 소개합니다.

## 설치

* 아래의 workflow를 추가합니다.

```yaml
version: 2
updates:
- package-ecosystem: npm
  directory: "/"
  schedule:
    interval: daily
    time: "20:00"
```

위의 설정은 npm 패키지 업데이트를 루트 디렉토리에서 실행하는데 매일 20:00시에 수행하며 최대 PR의 갯수는 10개로 제한하는 설정입니다.

### `package-ecosystem`

`package-ecosystem` 의 경우엔 npm 이외에도 여러가지 package manager를 지원합니다.

![package_manager](https://i.imgur.com/j6Jzv6l.png)

### `directory`

`directory` 의 경우엔 패키지 설정 파일이 들어있는 디렉토리를 지정해 주는 설정입니다.

npm의 경우엔 `package.json` 이 위치한 경로를 입력해주면 됩니다.

### `schedule`

`schedule`은 업데이트를 체크할 주기를 결정합니다.

```yaml
  schedule:
    interval: daily
  schedule:
    interval: weekly
  schedule:
    interval: monthly
```

위와 같이 daily, weekly, monthly의 3가지 옵션이 존재합니다.

또한 특정 시간에 업데이트를 실행하는 경우도 있습니다. 이는 time, timezone으로 설정이 가능합니다.

```yaml
  schedule:
    interval: daily
    time: "09:00"
    timezone: "Asia/Tokyo"
```

### `ignore`

`ignore`는 업데이트 체크 여부를 하지 않을 dependency를 지정합니다.

```yaml
# Use `ignore` to specify dependencies that should not be updated 

version: 2
updates:
  - package-ecosystem: "npm"
    directory: "/"
    schedule:
      interval: "daily"
    ignore:
      - dependency-name: "express"
        # For Express, ignore all updates for version 4 and 5
        versions: ["4.x", "5.x"]
        # For Lodash, ignore all updates
      - dependency-name: "lodash"
```
위와 같이 패키지명을 지정하여 통째로 무시할 수 있으며, 패키지명과 버전을 명시해서 특정 버전대만을 무시할 수도 있습니다.

### `open-pull-requests-limit`

`open-pull-requests-limit`는 Dependabot이 Open할 최대 Pull Request의 갯수를 지정합니다. 만약 Dependabot이 해당 제한만큼 Pull Request를 생성했다면 해당 PR들을 Close하거나 Merge하기 전 까지 새로운 PR을 생성하지 않습니다.

```yaml
# Specify the number of open pull requests allowed

version: 2
updates:
  - package-ecosystem: "npm"
    directory: "/"
    schedule:
      interval: "daily"
    # Disable version updates for npm dependencies
    open-pull-requests-limit: 0

  - package-ecosystem: "pip"
    directory: "/"
    schedule:
      interval: "daily"
    # Allow up to 10 open pull requests for pip dependencies
    open-pull-requests-limit: 10
```

이후로도 여러가지 옵션이 있지만 나머지는 [링크](https://docs.github.com/en/code-security/supply-chain-security/keeping-your-dependencies-updated-automatically/configuration-options-for-dependency-updates#configuration-options-for-updates)에서 확인해 보세요

## Auto-Dependabot

위에서 Dependabot이 Pull Request를 열어주었을 때 해당 PR이 Github Action들을 전부 통과했다면 해당 PR을 자동으로 Merge하게 됩니다.

### 설치

아래 workflow를 통해서 설정할 수 있습니다. 여기엔 SECRET값으로 repo를 컨트롤할 수 있는 권한을 가진 Github Token이 필요합니다.

```yaml
name: auto-merge

on:
  pull_request_target:

jobs:
  auto-merge:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v2
      - uses: ahmadnassri/action-dependabot-auto-merge@v2
        with:
          target: minor
          github-token: ${{ secrets.SECRET }}
```

위 workflow를 적용했다면 아래 사진처럼 모든 Checks에 대해서 Pass를 한 Pull Request에 대해서 자동으로 Review를 설정하고  자동으로 Merge를 하게 됩니다.

![ss](https://i.imgur.com/6UrvNmO.png)

