---
layout: post
title: "Husky + Github Action + commitlint을 사용하여 commit message 관리하기"
date: "2021-08-25 23:03:44 +0900"
tags: [tutorial]
---

![ss](https://commitlint.js.org/assets/commitlint.svg)

`commitlint` 는 commit message를 linting해주는 모듈입니다. 이를 `husky` 혹은 `github action과` 함께 사용한다면 더욱 편리하게 commit message를 관리할 수 있습니다.

https://commitlint.js.org/#/

### 설치

```bash
yarn add @commitlint/cli @commitlint/config-conventional -D
```

### configure

```bash
echo "module.exports = {extends: ['@commitlint/config-conventional']}" > commitlint.config.js
```

---

### Husky를 이용한 Commitlint 적용법

```json
    "husky": {
        "hooks": {
            "commit-msg": "commitlint -E HUSKY_GIT_PARAMS"
        }
    }

```

위와 같이 husky를 설정하게 되면 `git commit` 수행 시 커밋 메시지를 불러와 commitlint를 수행하게 됩니다.

```json
    "husky": {
        "hooks": {
            "commit-msg": "if git-branch-is dev; then commitlint -E HUSKY_GIT_PARAMS; fi"
        }
    },

```
또한 `git-branch-is` 와 결합하여 특정 브랜치에서만 commitlint가 수행되게 할 수도 있습니다.

위 명령어는 `dev` 브랜치에 대해서만 `commitlint`를 수행하게 하는 명령어 입니다.

### Github Workflow를 이용한 Commitlint 적용법

`Github Action을` 사용하여 `commitlint`를 수행할 수 있습니다.

이러한 Workflow를 사용하여 PR시 commit convention을 준수하지 않은 PR에 대해 merge를 막는다거나 하는 방법으로 사용할 수 있습니다.

```yaml
# Run commitlint on Pull Requests and commits
name: commitlint
on:
  pull_request:
      types: ['opened', 'edited', 'reopened', 'synchronize']
  push:
      branches:
        - '**'

jobs:
  lint-pull-request-name:
    # Only on pull requests
    if: github.event_name == 'pull_request'
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v1
      - run: yarn add @commitlint/config-conventional
      - uses: JulienKode/pull-request-name-linter-action@v0.1.2
  lint-commits:
    # Only if we are pushing or merging PR to the master
    if: (github.event_name == 'pull_request' && github.base_ref == 'refs/heads/master') || github.event_name == 'push'
    runs-on: ubuntu-latest
    env:
      GITHUB_TOKEN: ${{ secrets.SECRET }}
    steps:
      - uses: actions/checkout@v2
        with:
          fetch-depth: 30 # Its fine to lint last 30 commits only
      - run: yarn add @commitlint/config-conventional
      - uses: wagoid/commitlint-github-action@v1
```