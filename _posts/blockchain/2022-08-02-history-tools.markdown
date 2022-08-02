---
layout: post
title: "EOSIO History-tools 구축하기"
date: "2022-08-02 13:03:44 +0900"
tags: [blockchain]
---

![title](https://i.imgur.com/vwv6jRN.png){: width="100%" height="100%"}

## About

[Homepage](https://eos.io/for-developers/build/history-tools/)

[Github](https://github.com/EOSIO/history-tools)

History Tools는 EOSIO의 State History Plugin을 사용하여 Postgres등 RDB에 블록체인 데이터를 저장하는 솔루션 입니다.

기존 History Plugin을 사용하면 RDB에 저장하지 않고 블록체인 노드에서 자체적으로 데이터를 호출 할 수 있었으나 해당 Plugin이 Deprecated 되어 History-tools를 사용하여 비슷한 기능을 하게 끔 구축할 수 있습니다.

## Prerequisite

본 문서에서는 아래와 같은 솔루션을 사용하여 구축합니다.

* EOSIO Node
* EOSIO History-tools
* Postgres
* Hasura (Optional)

현재 History-tools 최신버전 (1.0.0)에는 postgres만을 지원하고 있기 때문에 다른 DB는 사용하지 않습니다.

또한 모든 솔루션은 Docker를 이용하여 구축 할 예정입니다.

## Setup

### Node Setup

[state_history_plugin](https://developers.eos.io/manuals/eos/v2.0/nodeos/plugins/state_history_plugin/index)

EOSIO를 구축할 때 반드시 State History Plugin을 활성화 하여 구축을 해 주어야 합니다.

본 문서에서는 EOSIO 구축에 대해서는 자세하게 설명하지 않겠습니다.

### Postgres + Hasura Setup

```yaml
# docker-compose.yaml
version: '3'
services:
  ship-postgres:
    container_name: eosio-history-postgres
    image: postgres:13
    ports:
      - "5432:5432"
    environment:
        POSTGRES_DB: postgres
        POSTGRES_PASSWORD: postgres
  graphql-engine:
      image: hasura/graphql-engine
      container_name: hasura
      ports:
      - "8080:8080"
      depends_on:
      - ship-postgres
      restart: always
      environment:
        HASURA_GRAPHQL_DATABASE_URL: postgres://postgres:postgres@ship-postgres:5432/postgres
        HASURA_GRAPHQL_ENABLE_CONSOLE: "true"
        HASURA_GRAPHQL_DEV_MODE: "true"
        HASURA_GRAPHQL_ADMIN_SECRET: myadminsecretkey
        HASURA_GRAPHQL_UNAUTHORIZED_ROLE: anonymous
```

위 yaml 파일은 postgres + hasura를 구축하기 위한 docker-compose 파일 입니다.

위 텍스트를 docker-compose.yaml로 저장하고 아래 명령어를 실행하여 줍니다.

```sh
docker-compose up -d
```

![ss](https://i.imgur.com/PJL46E5.png)

### History tools Setup

```yaml
version: '3'
services:
  ship-history-tools:
    image: sweatpotato/history-tools:0.3.0-alpha
    container_name: history-tools
    environment:
      PGUSER: postgres
      PGPASSWORD: ${POSTGRES_PASSWORD}
      PGDATABASE: ${POSTGRES_DB}
      PGHOST: ${POSTGRES_HOST}
      PGPORT: ${POSTGRES_PORT}
    command: fill-pg --plugin fill_plugin --fill-connect-to ${EOSIO_SHIP_HOST}:${EOSIO_SHIP_PORT} --fill-skip-to 0 --plugin fill_pg_plugin --fill-trx "-:executed::eosio:onblock" --fill-trx "+:executed:::" --fpg-drop --fpg-create
```

History tools도 마찬가지로 위의 docker-compose.yaml 파일을 사용하여 구축합니다.

Postgres 환경변수값과 Command의 EOSIO 노드의 Host 및 State History Plugin PORT 값을 적절히 넣어 실행합니다.

```sh
docker-compose up -d
```

![ss](https://i.imgur.com/cZrWecj.png)

로그를 조회해 본다면 History-tools에서 블록 단위로 싱크를 받고 있는 것을 확인할 수 있습니다.

Hasura를 확인한다면 테이블 및 데이터 구조도 확인이 가능합니다.

## 마무리
이 글에서 EOSIO에 History-tools를 붙여 RDB에 데이터를 저장해 보았습니다. EOSIO 프로젝트에서 가장 간단한 블록체인 데이터 파싱 툴이지만 RDB를 사용하는 만큼 블록이 많아지면 많아 질수록 데이터 조회에 대해서 속도가 느려지는 것을 체감하고 있습니다. 

이 부분은 이전 포스트에 설명 드렸던 Hyperion을 통해서 어느정도 해소가 될 수 있다고 생각합니다. 혹은 deprecated긴 하지만 아직 History Plugin도 사용할 수 있으니 데이터를 끌어오는 여러가지 방법 중 하나를 선택하여 구축 할 수 있습니다.