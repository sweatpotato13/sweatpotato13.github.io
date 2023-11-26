---
layout: post
title: "체인링크 CCIP란?"
date: "2023-11-26 13:03:44 +0900"
tags: [blockchain]
---

# Chainlink CCIP

**Chainlink Cross-Chain Interoperability Protocol (CCIP)** 는 EVM 기반 체인에서 Cross-Chain Interoperability를 제공하는 프로토콜 입니다.

체인링크 CCIP는 아래와 같은 기능을 지원합니다.

- *Arbitrary Messaging*: 임의의 데이터를 다른 네트워크의 Smart contract로 보낼 수 있습니다. 
 
- *Token Transfer*: 토큰을 다른 네트워크의 Smart contract 혹은 이더리움 주소로 전송할 수 있습니다.

- *Programmable Token Transfer*: 단일 트랜잭션 내에서 임의 데이터와 토큰을 동시에 전송할 수 있습니다.
-

# Architecture 

![architecture](https://docs.chain.link/images/ccip/basic-architecture.png)

위 사진은 CCIP의 기본적인 아키텍쳐를 보여줍니다. 

Router는 Smart contract 형식으로 구현되어 있으며 아래와 같은 기능을 수행할 수 있습니다.

- 다른 네트워크의 Smart contract transaction을 호출할 수 있습니다. 
- 다른 네트워크의 Smart contract 혹은 이더리움 주소로 토큰을 전송할 수 있습니다.
- 동일한 트랜잭션으로 토큰과 임의의 메세지를 함께 보낼 수 있습니다.

## Component

![images](https://docs.chain.link/images/ccip/ccip-diagram-04_v04.webp)

### Onchain components

#### Router

Router는 기본 CCIP의 기본적인 smart contract이며 체인당 하나의 Router contract가 존재합니다.

Router contract는 명령을 OnRamp로 라우팅하는 역할을 수행합니다. 

#### Commit store

[Committing DON](#committing-don)은 `CommitStore` contract와 상호작용하여 메시지의 Merkle root를 출발지 네트워크에 저장합니다.

Merkle root는 [Risk Management Network](#risk-management-network) 에 의해 블레싱(blessing) 되어야 [Executing DON](#executing-don)이 이를 목적지 네트워크에서 실행할 수 있습니다.

`CommitStore`은 메시지가 Risk Management Network에 의해서 blessing 되었고 레인(Lane)당 하나씩 존재하는지 체크합니다.

여기서 레인이란 출발지 네트워크와 목적지 네트워크간 경로를 의미합니다. 레인은 단발향이며 A->B 와 B->A는 별개의 레인입니다.

#### OnRamp

OnRamp contract는 레인당 하나씩 존재합니다. 이 컨트랙트는 아래와 같은 기능을 수행합니다.

- 전송하려는 내용이 목적지 네트워크에 맞는지 검증합니다.

- Message size limit, gas limits을 확인합니다.

- 수신자의 메시지 순서를 보장하기 위해 메시지 순서를 관리합니다.

- 수수료를 관리합니다.

- 메시지에 토큰이 포함되어 있는경우 [Token pools](#token-pools)와 상호작용 합니다.

- Committing DON이 모니터링 할 수 있도록 이벤트를 내보냅니다.

#### OffRamp

OffRamp contract는 레인당 하나씩 존재합니다. 이 컨트랙트는 아래와 같은 기능을 수행합니다.

- Executing DON이 제공한 Proof를 사용하여 실제로 메시지가 커밋 되었고, Blessing 되었는지 확인합니다.

- 트랜잭션이 정확하게 한번만 실행 되었는지 확인합니다.

- 검증 이후 수신 된 모든 메시지를 Router로 전송합니다. 이 과정에서 토큰 전송이 포함된 경우 OffRamp는 TokenPool을 호출하여 토큰을 올바른 수신자에게 전송합니다. 

#### Token pools

각각에 토큰은 OnRamp 혹은 OffRamp가 ERC-20에 대한 기능을 원활하게 하기 위한 token pool을 가지고 있습니다. 

네이티브 토큰(eg. ETH, MATIC)의 경우엔 해당 기본 체인에서만 발행될 수 있기 때문에 Wrapped Token의 형식으로 목적지 네트워크에서 발행 됩니다.

총 발행량이 고정된 토큰에 경우 Lock and Mint의 방식을 사용하고 

일반적으로 총 발행량이 고정되어 있지 않은 토큰에 경우 Burn and Mint의 방식을 사용합니다. 

#### Risk Management Network contract

bless 혹은 curse에 대한 내역을 관리합니다. 

### OffChain components

#### Committing DON

Committing DON은 출발지 네트워크와 목적지 네트워크간 트랜잭션을 모니터링하는 작업을 수행합니다.

- 출발지 네트워크에서 OnRamp contract의 이벤트를 모니터링 합니다.

- 출발지 네트워크에서 블록이 finalize 되는것을 기다립니다.

- 트랜잭션을 묶어서 Merkle root를 생성합니다.

- 목적지 네트워크의 CommitStore contract에 Merkle root를 저장힙니다. 

#### Executing DON

Executing DON은 출발지 네트워크와 목적지 네트워크간 트랜잭션을 실행하는 작업을 수행합니다.

- 출발지 네트워크의 OnRamp contract의 이벤트를 모니터링 합니다.

- 트랜잭션이 CommitStore contract에 존재하는 Merkle root의 일부인지 확인합니다.

- Risk Management Network에 의해 메시지가 blessing 될 떄 까지 기다립니다.

- Merkle root에 대하여 유효한 Merkle proof를 만들어 OffRamp contract를 통하여 트랜잭션을 실행합니다. 

#### Risk Management Network

Risk Management Network는 Commitiing DON이 Commit Store에 커밋한 Merkle root를 모니터링하는 독립된 노드 집합입니다.

각 노드는 커밋된 Merkle root를 OnRamp contract에서 받은 트랜잭션과 비교합니다. 

검증이 통과 되면 Risk Management Network contract를 이용하여 해당 Merkle root를 `bless` 합니다.

충분한 blessing이 있으면 해당 Merkle root를 실행할 수 있습니다.

만약 이상이 발생하는 경우 해당 시스템은 `curse` 합니다.
