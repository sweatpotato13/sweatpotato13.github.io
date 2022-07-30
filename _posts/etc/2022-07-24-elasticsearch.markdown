---
layout: post
title: "ElasticSearch에 대하여"
date: "2022-07-24 13:03:44 +0900"
tags: [etc, backend]
---
# ElasticSearch

![title](https://i.imgur.com/bGFBxti.png){: width="100%" height="100%"}

# About

최근 백엔드 개발에서 elasticserch의 활용이 늘어나고 있습니다. 이에 기존 RDB와 elasticsearch의 차이점과 어떤 면에서 elasticsearch가 이득을 볼 수 있는지 본 문서에서 정리 해 보도록 하겠습니다.

# What is elasticsearch

Elasticsearch는 Apache Lucene( 아파치 루씬 ) 기반의 Java 오픈소스 분산 검색 엔진입니다.

Elasticsearch를 통해 루씬 라이브러리를 단독으로 사용할 수 있게 되었으며, 방대한 양의 데이터를 신속하게, 거의 실시간( NRT, Near Real Time )으로 저장, 검색, 분석할 수 있습니다.

Elasticsearch는 검색을 위해 단독으로 사용되기도 하며, ELK( Elasticsearch / Logstatsh / Kibana )스택으로 사용되기도 합니다.

ELK 스택이란 다음과 같습니다.

- **Logstash**
    - 다양한 소스( DB, csv파일 등 )의 로그 또는 트랜잭션 데이터를 수집, 집계, 파싱하여 Elasticsearch로 전달
- **Elasticsearch**
    - Logstash로부터 받은 데이터를 검색 및 집계를 하여 필요한 관심 있는 정보를 획득
- **Kibana**
    - Elasticsearch의 빠른 검색을 통해 데이터를 시각화 및 모니터링

![ELK](https://i.imgur.com/Kon4H49.png){: width="100%" height="100%"}

# Architecture

![Architecture](https://i.imgur.com/gex3jOG.png){: width="100%" height="100%"}

- Cluster
    
    Elasticsearch에서 가장 큰 시스템 단위이며 node들의 집합입니다.
    
- Node
    
    Elasticsearch를 구성하는 하나의 단위 프로세스 입니다.
    
    다수의 shard로 구성되며 역할에 따라 Master node, Data node, Ingest node, Tribe node로 구분합니다.
    
- Index
    
    Elasticsearch 에서는 단일 데이터 단위를 도큐먼트(document) 라고 하며 이 도큐먼트를 모아놓은 집합을 인덱스(Index) 라고 합니다. 
    
- Shard
    
    인덱스는 기본적으로 샤드(shard)라는 단위로 분리되고 각 노드에 분산되어 저장이 됩니다. 샤드는 루씬의 단일 검색 인스턴스 입니다. 처음 생성된 샤드를 프라이머리 샤드(Primary Shard), 복제본은 리플리카(Replica) 라고 부릅니다. 같은 샤드와 복제본은 동일한 데이터를 담고 있으며 반드시 서로 다른 노드에 저장이 됩니다.
    

# Difference of Elasticsearch & RDB

일반적으로 RDB는 Row를 기반으로 데이터를 저장하고 있습니다. 그에 반해서 Elasticsearch는 index를 기반으로 하여 데이터를 저장하고 있습니다.

이러한 구조의 특성 때문에 RDB는 데이터 CRUD에서 편의성과 속도면에서 강점이 있지만 다양한 데이터를 필터링하고 검색하는데는 상대적으로 속도가 느립니다.

하지만 Elasticsearch는 데이터의 검색부분에서는 RDB보다 훨씬 빠르게 처리할 수 있지만 Update, Delete 에 있어서는 RDB보다 훨씬 많은 코스트를 소모할 수 있습니다.

또한 Elasticsearch는 REST API를 통해서 CRUD를 진행할 수 있다는 점에서 확장에 용이하다고 할 수 있습니다. (물론 RDB의 경우에도 Hasura와 같은 별도 툴을 이용하면 REST API로 CRUD를 진행할 순 있습니다.)

![Untitled](https://i.imgur.com/5GDpFiX.png)

# 마무리

이 글에서 elasticsearch에 대한 간단한 내용과 RDB와의 차이점을 알아보았습니다. elasticsearch가 유용한 툴은 맞지만 아직 RDB를 100% 대체하기엔 조금 힘들지 않나 하는 생각이 듭니다. 프로젝트의 성격에 따라서 RDB와 ES를 유용하게 사용한다면 더욱 효율적인 개발을 할 수 있을 것이라 생각합니다.
