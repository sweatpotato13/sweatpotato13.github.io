---
layout: post
title: "AntelopeIO (former EOSIO) hyperion 구축하기"
date: "2022-10-20 13:03:44 +0900"
tags: [blockchain]
---

![title](https://i.imgur.com/sCENLAJ.png){: width="100%" height="100%"}

## About

[Github](https://github.com/sweatpotato13/hyperion-history-api-docker)

EOSIO가 AntelopeIO로 리브랜딩 되면서 여러 프로젝트가 아카이빙 되었습니다. 이전에 소개해 드렸던 [history-tools](https://github.com/EOSIO/history-tools)도 그 중 하나입니다. 사실 history-tools를 사용하면서 DB에 데이터가 많아지게 되면 조회 성능이 크게 떨어져 불편한 경우가 많았습니다. 

본 글에서는 hyperion을 docker를 사용해서 구축하는 방법을 알아보겠습니다.


## Prerequisite

본 문서에서는 아래의 툴을 사용하여 구축합니다.

* Redis
* RabbitMQ
* Elasticsearch
* kibana
* Hyperion (3.3.5)

또한 모든 솔루션은 Docker를 이용하여 구축 할 예정입니다.

## Setup

### Node Setup

[state_history_plugin](https://developers.eos.io/manuals/eos/v2.0/nodeos/plugins/state_history_plugin/index)

EOSIO를 구축할 때 반드시 State History Plugin을 활성화 하여 구축을 해 주어야 합니다.

본 문서에서는 EOSIO 구축에 대해서는 자세하게 설명하지 않겠습니다. 

### Hyperion Config Setup

[Github](https://github.com/sweatpotato13/hyperion-history-api-docker) 내에 존재하는 connections.json 파일을 수정해 주어야 합니다. 본 문서에서는 docker 기반으로 기타 필요한 서비스를 셋업하기 때문에 chain endpoint만 수정해 주면 됩니다.

http에는 node http endpoint를, ship에는 node state_history_plugin endpoint를 입력해 주면 됩니다.

![ss](https://i.imgur.com/4oliWZI.png)

### Other Infrastructure Setup

```yaml
# docker-compose.yaml
version: '3.3'

services:
  redis:
    container_name: redis
    image: redis:alpine
    restart: on-failure
    networks:
      - hyperion

  rabbitmq:
    container_name: rabbitmq
    image: rabbitmq:alpine
    restart: on-failure
    environment:
      - RABBITMQ_DEFAULT_USER=username
      - RABBITMQ_DEFAULT_PASS=password
      - RABBITMQ_DEFAULT_VHOST=/hyperion
    ports:
      - 15672:15672
    networks:
      - hyperion

  elasticsearch:
    container_name: elasticsearch
    image: docker.elastic.co/elasticsearch/elasticsearch:7.17.6
    restart: on-failure
    environment:
      - discovery.type=single-node
      - cluster.name=es-cluster
      - node.name=es01
      - bootstrap.memory_lock=true
      - xpack.security.enabled=true
      - "ES_JAVA_OPTS=-Xms2g -Xmx2g"
      - ELASTIC_USERNAME=elastic
      - ELASTIC_PASSWORD=password
    ports:
      - 9200:9200
    networks:
      - hyperion

  kibana:
    container_name: kibana
    image: docker.elastic.co/kibana/kibana:7.17.6
    restart: on-failure
    environment:
      - ELASTICSEARCH_HOSTS=http://elasticsearch:9200
      - ELASTICSEARCH_USERNAME=elastic
      - ELASTICSEARCH_PASSWORD=password
    ports:
      - 5601:5601
    networks:
      - hyperion
    depends_on:
      - elasticsearch

  hyperion-indexer:
    container_name: hyperion-indexer
    image: sweatpotato/hyperion-history-api:3.3.5
    restart: on-failure
    depends_on:
      - elasticsearch
      - redis
      - rabbitmq
    volumes:
      - ./config/connections.json:/hyperion-history-api/connections.json
      - ./config/ecosystem.config.js:/hyperion-history-api/ecosystem.config.js
      - ./config/chains/:/hyperion-history-api/chains/
      - ./scripts/:/home/hyperion/scripts/
    networks:
      - hyperion
    command: bash -c "/home/hyperion/scripts/run-hyperion.sh ${SCRIPT:-false} eos-indexer"

  hyperion-api:
    container_name: hyperion-api
    image: sweatpotato/hyperion-history-api:3.3.5
    restart: on-failure
    ports:
      - 7000:7000
    depends_on:
      - hyperion-indexer
    volumes:
      - ./config/connections.json:/hyperion-history-api/connections.json
      - ./config/ecosystem.config.js:/hyperion-history-api/ecosystem.config.js
      - ./config/chains/:/hyperion-history-api/chains/
      - ./scripts/:/home/hyperion/scripts/
    networks:
      - hyperion
    command: bash -c "/home/hyperion/scripts/run-hyperion.sh ${SCRIPT:-false} eos-api"

networks:
  hyperion:
    driver: bridge
```

위 yaml 파일은 [Github](https://github.com/sweatpotato13/hyperion-history-api-docker) 에 있는 docker-compose yaml 파일 입니다.

위 텍스트를 docker-compose.yaml로 저장하고 아래 명령어를 실행하여 줍니다.

```sh
docker-compose up -d
```

![ss](https://i.imgur.com/mskl2MS.png)


### 결과

![indexer](https://i.imgur.com/yxi2XmM.png)

![api](https://i.imgur.com/W8zMudS.png)

위와 같이 hyperion-indexer와 hyperion-api 로그를 확인하였을 때 log가 정상적으로 출력된 것을 확인할 수 있습니다.

## 마무리
이 글에서 Hyperion을 구축하여 보았습니다. 확실히 history-tools를 사용했을 때 보다 조회에서 성능이 더 잘 나온다는것을 직접 체감할 수 있을 정도의 차이가 있었습니다. 또한 AntelopeIO로 리브랜딩 된 이후 버전에서의 node에서는 history-tools를 지원하지 않는 문제도 있기때문에 이제는 history-tools를 사용하기는 어려운 것 같습니다. hyperion은 아직 활발하게 개발되는 프로젝트이기 때문에 앞으로 더 기대 되는 프로젝트 입니다.