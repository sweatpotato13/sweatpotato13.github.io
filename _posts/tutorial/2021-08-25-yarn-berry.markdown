---
layout: post
title: "Yarn 2 (Yarn berry)로 node-module에서 해방되기"
date: "2021-08-25 23:03:44 +0900"
tags: [tutorial]
---

![Untitled](https://i.imgur.com/816lUTi.png)

## Description

node 기반으로 개발을 하게된다면 `node-module` 을 빼 놓을 수 없다. 하지만 `node-module`은 Ryan Lienhart Dahl도 "It's my fault and I'm very sorry." 라고 표현할 만큼 후회하고 있다고 하였다. [(링크)](https://tinyclouds.org/jsconf2018.pdf) 이 말대로 `node-module`은 비효율적인 의존성 관리때문에 많은 개발자들에게 놀림거리가 되고는 한다. 대표적인 예가 아래의 사진이다. 

![Untitled](https://i.imgur.com/Saej47O.png)

그렇다면 이러한 구조가 Yarn 2(Yarn berry) 에서는 어떻게 변경되었나 알아보자

기존 Yarn에서는 Hoisting 방식을 사용하여 가능한 모든 의존성 모듈을 중앙의 위치로 추출하고 병합합니다.

![Untitled](https://i.imgur.com/jdDE8Ca.png)

Hoisting을 사용하여 B@2.0을 유지하고 동일한 경로를 유지하면서 A@1.0, B@1.0 의 중복은 제거할 수 있었습니다. 

하지만 이러한 방법을 사용하게 되면 **유령 의존성(Phantom dependencies)** 이 발생하게 되는데. 어떠한 패키지가 Hoisting을 통해 최상단에 위치하게 되면 package에 추가하지 않고도 사용이 가능해지는 문제가 발생합니다. 

![Untitled](https://i.imgur.com/JYU7kTc.png)

이를 `Yarn 2` 에서는 PnP(Plug and Play) 를 통해서 해결하고자 하였습니다. 

`Yarn 2` 에서는 `yarn install`을 수행하게 되면 `node-module`이 생성되는 대신 `.pnp.cjs` 파일이 생성됩니다. 또한 실제 의존성 모듈은 `.yarn/cache`에 저장됩니다. `.pnp.cjs`에 명시된 정보에 따라 각 패키지들이 참조 되기 때문에 기존 처럼 `node-module`을 뒤져볼 필요가 사라져 속도가 증가하였습니다.

이렇게 의존성을 `.yarn/cache`에 저장해놓기 때문에 `.yarn` 폴더 전체를 git에 푸시한다면 `yarn install` 없이 바로 프로젝트를 구동할 수 있습니다. 

## Migration

1. `Yarn`을 설치 합니다

    ```
    npm install -g yarn
    ```

2. 프로젝트 경로로 이동하여 버전을 변경합니다

    ```
    yarn set version berry
    ```

3. `Yarn install` 을 수행하여 lock 파일에 대한 마이그레이션 을 수행합니다.
4. `.gitignore`에 아래와 같은 문구를 추가합니다.

    ```bash
    #If you're using Zero-Installs:
    .yarn/*
    !.yarn/cache
    !.yarn/patches
    !.yarn/plugins
    !.yarn/releases
    !.yarn/sdks
    !.yarn/versions

    #If you're not using Zero-Installs:
    .yarn/*
    !.yarn/patches
    !.yarn/releases
    !.yarn/plugins
    !.yarn/sdks
    !.yarn/versions
    .pnp.*
    ```

### Known Problem

1. `Yarn 2` 에는 아직 `yarn publish` 명령어 대신 `yarn npm publish` 를 사용해야합니다.
2. node를 사용한다면 node를 실행할 때 `yarn node` 를 사용해야 실행이 됩니다.