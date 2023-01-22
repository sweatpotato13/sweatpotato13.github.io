---
layout: post
title: "Polkadot 개념 정리 basics"
date: "2023-01-21 13:03:44 +0900"
tags: [blockchain]
---

본 문서는 Polkadot wiki를 읽고 나름대로 정리한 문서입니다. Polkadot에 대한 기본 이해가 부족한 상태에서 읽고 작성한 문서이기 때문에 본문에 오류가 있을 수 있습니다. 실제 문서는 [Learn](https://wiki.polkadot.network/docs/learn-accounts) 부터 시작되는 Basics 부분을 참고하세요.

## Introduction

Polkadot은 최초의 Fully-sharded Blockchain 입니다.

Polkadot은 `sharded model` 이라고 하는 네트워크 모델을 사용하고 있습니다.

Polkadot에는 시스템의 메인 체인처럼 동작하는 `Relay Chain` 이 있으며 WASM으로 컴파일 되고 Relay Chain API를 지원한다면 Polkadot에 `Parachain` 으로 붙어 동작할 수 있습니다.

![Untitled](https://i.imgur.com/pplJA1R.png)

Polkadot은 Bitcoin, Ethereum 등 체인과 양방향 상호작용을 위한 Bridge Parachain도 존재합니다. 이는 다른 Parachain간 트랜잭션 전송을 가능하게 합니다.

XCM (Cross-Consensus Messaging Format) 을 사용하여 서로 다른 Parachain간 메시지를 보낼 수 있습니다.

## Accounts

### Polkadot Accounts

Account는 Public, Private 부분으로 나누어져 있다.

Address는 Public한 부분이며 Key는 Private한 부분이다.

Polkadot은 `Ed25519 Sr25519 secp256k1` 커브를 지원한다.

Polkadot은 Multisig-account / anonymous proxy를 지원한다.

대부분의 Wallet은 니모닉 구문을 지원하며 아래와 같은 모습을 보인다.

```
'caution juice atom organ advance problem want pledge someone senior holiday very'
```

```
Secret seed (Private key): 0x056a6a4e203766ffbea3146967ef25e9daf677b14dc6f6ed8919b1983c9bebbc
Public key (SS58): 5F3sa2TJAWMqDhXG6jhV4N8ko9SxwGy8TpaNS1repo5EYjQX
```

Polkadot 기본 Address format은 `MultiAddress` 타입으로 이는 같은 니모닉구문으로 다른 Parachain에서 키를 생성할 수 있음을 의미합니다.

Polkadot은 여러 유형의 Balance가 있다.

- transferrable: 자유롭게 송금할 수 있는 잔액
- vested: 특정 시간에 정해진 일정에 따라 릴리즈 되는 토큰, 즉 잠겨있지만 특정 블록 수 이후에 해금됨
- locked: bonded,democracy, vested 중 가장 큰 잔액
- reserved: bonded,democracy, vested 이외의 목적으로 동결된 토큰 수
- bonded: 스테이킹을 위해 잠겨진 토큰
- redeemable: 잠금 해제가 가능하도록 준비된 토큰
- democracy: 온체인 참여를 위해 잠겨진 토큰

계정을 생성할 때 처음엔 오직 키만 생성하며 온체인에 등록되지 않습니다. 온체인에 등록하기 위해서 1 DOT을 예치하여야 합니다.

만약 예치금이 1 DOT 이하로 떨어지면 게정은 체인에서 삭제되지만 다시 해당 금액 이상으로 예치한다면 계정을 다시 사용할 수 있습니다.

Polkadot에 내장된 `Identities Pallet` 을 사용하면 계정에 대한 메타데이터를 등록할 수 있습니다.

### Account Generation

-

### Balance Transfers

Transfer를 수행할 수 있는 2가지 옵션이 존재

- `transfer keep-alive` : 예치금 (1 DOT) 미만 으로 송금하는걸 허용하지 않음
- `transfer` : 무시하고 보낼 수 있지만 예치금 미만으로 잔액이 떨어지면 계정이 회수됨

### **Extrinsics**

Polkadot은 [Substrate](https://substrate.io/) 를 이용하여 구축되었습니다.

Substrate는 Pallets라고 하는 모듈과 기타 라이브러리등을 제공합니다.

pallet에 권한이 있는경우 그 안에 포함된 기능을 호출하고 실행할 수 있습니다.

**Extrinsics** 은 3가지 타입이 존재합니다.

- signed transactions: 요청 보내는 계정의 서명이 포함된 트랜잭션
- unsigned transactions: 서명이 없기때문에 제출한 유저에 대한 정보가 없음
- inherents: 블록을 구축하는데 필요한 정보를 전달하는 블록 작성자가 만든 서명되지 않은 트랜잭션

### Multi-signature Accounts

Substrate 기반 체인에서는 Multisig 계정을 생성할 수 있습니다. 이러한 Multisig는 하나 이상의 주소와 threshold로 구성됩니다.

Multi-signature accounts는 생성 된 이후에 수정될 수 없습니다. 구성원이나 임계값을 변경하기 위해서는 다중 서명을 해체하고 새로운 다중 서명을 생성해야 합니다.

Multisig로 수행할 수 있는 작업은 3가지가 있습니다.

- asMulti: 다중서명 트랜잭션을 시작하거나 종료
- approveAsMulti: 트랜잭션을 승인하고 다음 서명자에게 넘김
- cancleAsMulti: 취소함

![Untitled](https://i.imgur.com/OL2IqD8.png)

### Proxy Accounts

Proxy Account는 다른 계정을 대신해서 기능을 수행할 수 있습니다.

Proxy account는 proxy pallet를 통해서 설정할 수 있습니다. proxy type은 다음과 같습니다.

- Any: 모든 거래 수행 가능
- Non-transfer: transfer를 제외한 모든 기능 수행 가능
- Governance: governance 관련 트랜잭션만 수행할 수 있도록 허용
- Staking: staking 관련 트랜잭션만 수행할 수 있도록 허용
- Identity Judgement: 계정 생성을 위한 신원에 대해 판단만 가능
- Auction: Parachain 경매 및 기타 트랜잭션만 허용

Proxy를 생성하기 위해서 토큰을 예치해야 합니다. (모든 peer에 복제되어야 하기때문에 스토리지를 사용하게 되므로 보증금 필요)

Time-delay를 제공하여 보안을 높일 수 있습니다. 이 시간 안에 프록시가 수행한 작업을 취소할 수 있습니다.

Anonymous proxy는 기본 계정에 의해서 생성되었지만 할당되진 않은 새 계정 입니다. 그리고 기본 계정은 Anonymous proxy를 대신하여 프록시의 역할을 합니다. 여기서 Anonymous proxy와 기본계정간 관계가 끊어지면 더이상 Anonymous proxy에 접근할 수 없습니다.

![Untitled](https://i.imgur.com/ZgEnqkO.png)

## Tokens and Assets

### Assets

Statemint Parachain은 Polkadot 네트워크에서 자산의 생성, 관리, 사용을 전문으로 하는 데이터 구조를 포함하고 있습니다.

**Fungible Assets**

10 DOT을 예치할 수 있다면 누구나 Statemint parachain에서 자산을 생성할 수 있습니다. 자산을 생성할 때 생성자는 자산을 식별하기 위해 unique한 `u32` 타입은 `AssetId`를 지정 해야 합니다.

Asset에는 여러가지 권한을 포함하고 있는데 아래와 같다

- Owner: 다른 세 가지 역할을 하는 계정을 설정하고 자산에 대한 메타데이터를 설정할 수 있다.
- Issuer: 토큰을 발행 및 소각할 수 있다.
- Admin: 강제 이체 및 계정을 동결 및 동결해제 할 수 있다.
- Freezer: 자산을 동결할 수 있다.

Statemint는 `approve_transfer`, `transfer_approved`, `cancel_approval` 인터페이스를 제공합니다. 이 인터페이스를 사용하여 사용자가 계정을 대신해서 transfer를 할 수 있도록 권한을 줄 수 있습니다.

Statemint는 reserve-backed 시스템을 이용하여 다른 Parachain간 자산 전송을 관리한다.

**Non-Fungible Assets**

Statemint의 Uniques pallet는 NFT를 나타냅니다.

100 DOT을 예치할 수 있다면 누구나 자산을 생성할 수 있습니다. 거버넌스가 free holding 으로 거버넌스를 설정하지 않으면 예치금이 필요합니다.

생성자는 자산을 식별하기 위해 위의 `AssetId` 와 유사한 역할을 하는`ClassId` 를 설정하여야 합니다.

또한 Fungible Assets과 마찬가지로 owner, Issuer, admin, freezer 권한을 설정할 수 있습니다.

NFT 자산에서는 메타데이터를 가질 수 있는데 이 메타데이터에는 IPFS 해시값 등의 메타데이터를 설정할 수 있습니다.

NFT 에서도 FT와 마찬가지로 `approve_transfer`, `transfer_approved`, `cancel_approval` 인터페이스를 사용하여 권한을 위임할 수 있습니다.

### DOT

DOT은 Polkadot 네트워크에서 사용되는 Native Token 입니다.

가장 작은 단위를 Planck라고 하며 0.0000000001 DOT 입니다.

DOT은 크게 3가지의 사용처를 가집니다.

- 네트워크의 거버넌스에 참여하기 위함
- 네트워크 운영을 위한 스테이킹
- parachain으로 polkadot에 체인을 연결하기 위해

### Statemint

Statemint는 일반적인 자산(FT, NFT)에 대한 기능을 제공하는 Parachain 입니다.

### Teleporting Assets

asset teleportation이란 자산을 다른 체인(Parachain) 간 이동하는 프로세스를 의미합니다.

- Initiate Teleport
  보내는 계정에서 teleport할 자산을 기록하여 꺼냅니다.
- Receive Teleported Assets
  보내는 쪽은 XCM의 `ReceivedTeleportedAssets` 을 통해 보내는 양 및 받는 계정을 포함하는 파라미터를 생성합니다.

- Deposit Assets
  받는 쪽에서 받는 계정으로 자산을 예치시킵니다.

## Compoments

### Architecture

**Components**

- Relay Chain
  - Polkadot의 중앙 체인
  - 모든 Validator는 Relay Chain에 DOT을 스테이킹을 필요로 한다.
  - 거버넌스 매커니즘, Parachain 경매, NPoS에 참여하는 등 적은수의 트랜잭션으로 구성되어 있다.
  - Relay chain은 최소한의 기능만 가지고 있습니다.
  - Smart Contract는 지원하지 않고 있으며 이는 다른 Parachain에 위임하고 있다.
- Parachain and Parathread Slots
  - Polkadot은 Parachain, Parathread 의 2가지 모델을 사용하고 있다.
  - Parachain에는 체인 전용 슬롯이 있어 지속적으로 실행되는 프로세스와 같으며
  - Parathread는 그룹간 슬롯을 공유하여 잠깐 실행되는 프로세스와 비슷합니다.
  - Parachain은 validators가 검증할 수 있는 Proof를 생성해야한다는 제약 이외에 다른제약은 없다.
  - 이 Proof은 Parachain의 State Transition을 검증합니다.
  - Parathread는 Parachain과 동일한 API를 사용합니다.
  - Parachain은 DOT을 스테이킹 해놓아야 하고, parathread는 블록 당 사용료를 지불해야 합니다.
  - Parachain은 Parathread가 될 수 있으며 역도 가능합니다.
- Shared Security
  - Relay chain에 붙는 모든 Parachain은 Relay chain의 Security를 공유합니다.
  - Polkadot은 Relay chain 및 모든 연결된 Parachain에 대해서 shared state를 가집니다.
  - Relay chain을 이전으로 되돌린다면 붙어있는 Parachain 또한 되돌아갑니다. 이는 전체 시스템에 대한 유효성이 지속되고 개별 부분이 손상되지 않도록 하기 위함입니다.

**Interoperability**

- XCM
  - Cross-Consensus Message
  - Protocol이 아닌 Format
  - 메시지의 구조화만 정의하고 있음
  - XCM 형식은 Parachain이 서로 통신할 수 있게 하는 메시지 포맷
  - XCMP는 이를 전달하는 매커니즘
- Bridges
  - 임의의 데이터를 네트워크에서 다른 네트워크로 전송할 수 있는 통로
  - 브릿지를 통해서 다른 네트위크는 상호 운용 가능하지만 각각 규칙이나 거버넌스를 독립적으로 가져갈 수 있습니다.
  - Polkadot 에서 Bridge는 Relay Chain에 연결되어 Collator가 유지관리하는 Polkadot 컨센서스에 의해서 관리됩니다.

**Main Actors**

- Validators
  - Validator가 Validator set으로 선출되면 Relay chain에서 블록을 생성합니다.
  - Collator로 부터 Proof를 승인 받으면 보상을 받습니다.
- Nominators
  - 특정 Validator에게 스테이킹 지분을 지원하여 Validator가 받는 보상을 일부 받습니다.
- Collators
  - Relay Chain, Parachain 양쪽 모드에서 Full node이다.
  - Parachain의 트랜잭션을 수집하여 Relay chain의 Validator을 위한 Proof를 생성합니다.
  - parachain 블록은 Collator가 생성하지만 Relay chain에서 Validator은 유효성 검증만 한다

### Collator

Collator는 Parachain 트랜잭션을 수집하고 State Transition Proof를 만들어 Relay chain validators에게 전달합니다.

Collator는 Parachain의 트랜잭션을 통해서 Parachain 블록을 구성하고 이 블럭을 기반으로 State Transition Proof을 만들어 Validator에게 제공함으로써 Parachain을 유지합니다.

Collator는 Parachain과 Relay Chain에 대해서 Full node를 유지해야 합니다.

Collator는 네트워크를 보호하지 않습니다. Parachain Block이 Invalid 하다면 그것은 Validator에 의해 거부될 것입니다.

State Transition Proof 가 2/3 이상의 Validator에 의해서 승인되었다면 그것은 backable 한것으로 간주합니다.

Validators는 아래 순서대로 검증을 수행해야 합니다.

- Candidate(State Transition Proof)는 persisted validation data의 매개변수를 초과하면 안된다.
- Collator의 서명이 검증되어야 한다.
- Parachain Runtime을 실행함으로써 Candidate를 검증한다.

Candidate 가 위의 조건을 충족한다면 선택된 Relay Chain block author가 각 Parachain의 backed Candidate 를 골라 Relay Chain Block에 포함시킨다.

**XCM**

Collator은 XCM의 핵심 요소 입니다.

Collator는 모든 프로세스가 끝날때 까지의 모든 트랜잭션을 수집합니다. 그 이후 Collator는 Parachain Block Candidate에 서명하고 State Transition Proofs를 생성합니다. Collator는 Candidate Block + State Transition Proofs 를 validators에 전달하고 validators는 Candidate Block 내 트랜잭션들을 검증합니다. 검증이 완료되면 Candidate Block를 Relay chain에 전송합니다.

### **Consensus**

**Nominated Proof of Stake**

Pool을 온체인으로 구현하여 토큰 보유자가 자신을 대표할 수 있는 Validator에게 투표할 수 있도록 합니다.

Polkadot은 이러한 Validator set을 선택하는 매커니즘으로 NPoS(Nominated Proof-of Stake)를 사용합니다. Validators는 BABE에서 새로운 블록을 생성하고 Parachain을 검증하는 역할을 수행합니다.

Polkadot의 컨센서스 프로토콜에는 2가지 프로토콜이 사용됩니다. (`GRANDPA`, `BABE`)

Polkadot은 하이브리드 컨센서스를 사용하는데 하이브리드 컨센서스에서는 블록 생성 프로세스에서finality 부분을 분리합니다.

**BABE(Blind Assignment for Blockchain Extension)**

BABE는 validator와 새로운 블록의 author 사이에서 동작하는 블록 생성 메커니즘 입니다.

BABE는 지분에 따라서 validator에게 블록 생성 슬롯을 할당합니다.

BABE의 실행은 epoch라고 하는 단계에서 이루어 집니다. 각 epoch는 미리 정의된 수의 슬롯으로 나뉘어 각 epoch의 모든 슬롯은 0부터 순차적으로 인덱싱 됩니다. 각 epoch가 시작될 때 BABE 노드는 Block-Production-Lottery 알고리즘을 실행하여 어느 슬롯에서 블록을 생성할지 결정하고 다른 블록 생성자에게 알립니다.

Validators는 모든 슬롯에 대한 추첨에 참여하며 각 슬롯에 대해 블록 생성자 후보 여부를 알려줍니다.

슬롯은 길이가 대략 6초인 개별 시간 단위입니다. Validators에게 슬롯을 할당하는 메커니즘은 랜덤을 기반으로 하기 때문에 여러 Validators가 동일한 slot의 후보가 될 수 있으며 slot이 비어있을 수 있어 블록시간이 항상 일정하지 않을 수 있습니다.

- **Multiple Validators per Slot[](https://wiki.polkadot.network/docs/learn-consensus#multiple-validators-per-slot)**
  특정 슬롯에서 여러 Validators가 블록 생성자 후보로 있는 경우 모두가 블록을 생성하고 이를 네트워크에 브로드캐스팅 합니다. 여기서 먼저 네트워크에 도달한 Validators가 최종적으로 선정됩니다.
- **No Validators in Slot[](https://wiki.polkadot.network/docs/learn-consensus#no-validators-in-slot)**
  슬롯에 Validators가 없는경우 백그라운드에서 RR Validators Selection 알고리즘이 실행되어 Validators가 선정되어 블록을 생성합니다.

**GRANDPA (GHOST-based Recursive ANcestor Deriving Prefix Agreement)**

GRANDPA는 Polkadot Relay Chain에 구현되는 finality gadget 입니다.

Polkadot 호스트는 GRANDPA를 사용하여 블록을 Finality 합니다. Finality는 Validator 노드의 투표에 의해 부여되며 Validator는 블록 생성과 이를 병행하여 실행합니다.

2/3 이상의 Validator가 특정 블록을 포함하는 체인을 검증하는 순간 해당 블록까지의 모든 블록이 한번에 Finalize 됩니다.

**BEEFY (Bridge Efficiency Enabling Finality Yielder)**

Polkadot 네트워크와 다른 블록체인 (Ethereum과 같은) 간 효율적인 브릿지를 지원히기 위한 GRANDPA의 보조 프로토콜 입니다. 이 프로토콜을 통해서 다른 네트워크에 있는 참여자들이 Polkadot Relay Chain의 Validator가 생성한 Finality Proof를 검증할 수 있습니다.

![Untitled](https://i.imgur.com/gsQpI5e.png)

### **Governance**

**Referenda**

Referenda는 간단한 Stake 기반의 투표 방식입니다. 각각의 referendum는 런타임에서 함수 호출의 형태를 하는 제안을 가지고 있습니다.

Referenda는 투표를 할 수 있는 정해진 기간이 있으며 투표가 승인되면 해당 함수의 호출이 이루어 집니다. 투표시 선택할 수 있는 옵션은 찬성, 반대, 기권입니다.

Referenda는 여러 방식으로 시작할 수 있습니다.

- 공개적으로 제출된 제안
- 과반수 이상의 동의로 Council에서 제출된 제안
- prior referendum 제정의 일환으로 제출된 제안
- Technical Committee가 제출하고 Councli이 승인한 긴급제안

누구나 최소한의 토큰을 예치 하는것으로 referendum를 제안할 수 있습니다. 누군가 제안에 동의한다면 동일한 양의 토큰을 예치할 수 있습니다. 예치금액이 높은 제안이 다음 투표에 안건으로 선정됩니다.

예치된 금액은 제안이 표결에 부쳐지면 락이 해제됩니다. 제안 대기열에는 최대 100개의 공개 제안이 있을 수 있습니다.

Unanimous Council - Council의 모든 멤버가 제안에 동의하면 전체투표로 옮길 수 있습니다.

Majority Council - Council의 과반수가 동의하는 경우 전체투표로 옮길 수 있습니다.

긴급 투표가 진행 중인 경우를 제외하고 활성화된 전체투표는 하나만 존재할 수 있습니다.

대기열에 제안이 존재하는 한 28일마다 새로운 Referenda가 활성화 됩니다. 대기열은 Council에서 승인된 제안을 위한 대기열이있고 공개적으로 제출된 제안에 대한 대기열 2가지가 존재합니다. Referenda는 두 대기열의 최우선순위가 번갈아 가면서 진행됩니다.

상위 제안은 Stake 양에 따라서 정해집니다. 두 대기열중 최상위 제안 중 Stake가 많이 되어있는 제안이 우선순위로 선정됩니다.

Referenda에 투표하기 위해서 voter는 투표 종료시 까지 토큰에 대해 Lock을 걸어야 합니다.

```
Example:

Peter: Votes No with 10 DOT for a 128 week lock period => 10 x 6 = 60 Votes

Logan: Votes Yes with 20 DOT for a 4 week lock period => 20 x 1 = 20 Votes

Kevin: Votes Yes with 15 DOT for a 8 week lock period => 15 x 2 = 30 Votes
```

집계 방식은 총 3가지 시나리오가 있을 수 있습니다.

- Public - Positive Turnout Bias (Super-Majority Approve)
- Council ( Complete ) - Negative Turnout Bias (Super-Majority Against)
- Council ( Majority ) - Simple Majority

투표 결과를 적용하기위해 아래 공식중 하나를 적용합니다.

Super-Majority Approve

![Untitled](https://i.imgur.com/xBHFOoy.png)

Super-Majority Against

![Untitled](https://i.imgur.com/EMdAtEe.png)

Simple Majority

![Untitled](https://i.imgur.com/88QgFkJ.png)

```
approve - the number of aye votes

against - the number of nay votes

turnout - the total number of voting tokens (does not include conviction)

electorate - the total number of tokens issued in the network
```

Polkadot은 Holder가 자발적으로 토큰을 잠글 수 있는 기간을 선언하여 투표권을 높일 수 있습니다.

| Lock Periods | Vote Multiplier | Length in Days |
| ------------ | --------------- | -------------- |
| 0            | 0.1             |                |
| 1            | 1               |                |
| 2            | 2               |                |
| 4            | 3               |                |
| 8            | 4               |                |
| 16           | 5               |                |
| 32           | 6               |                |

총 6단계로 설정되며 각 기간은 28일입니다.

Adaptive Quorum Biasing 라는 개념을 도입하여 명확한 다수가 없는경우에 제안이 통과되는것을 더 쉽게 혹은 더 어렵게 만드는데 효과적인 절대 다수를 변경하는데 사용할 수 있는 레버 역할을 합니다.

![Untitled](https://i.imgur.com/WrFciYu.png)

이미지를 예시로 들면 Turnout 25%의 경우 투표율이 25%이며 Positive Turnout Bias를 적용했을 경우 66%의 찬성을 받아야 합니다. 반대로 투표율이 75%일 경우 54%의 찬성을 획득하면 됩니다.

Negative Turnout Bias를 적용할 경우 투표율이 25%일때 찬성이 34%를 획득하면 되고 투표율이 75%일 경우 46%의 찬성을 받으면 됩니다.

**Council**

Council은 온체인 계정으로 구성된 엔티티입니다. Council는 13명의 구성원으로 구성되어 있습니다.

Council은 합리적인 투표를 제안하는것, 악의적인 제안으 취소하는것, 기술위원회 선출등의 임무룰 수행합니다.

기술위원회가 만장일치로 승인하거나, Roo가 Cancel을 트리거하는경우 제안은 취소될 수 있습니다. 취소된 제안의 보증금은 소각됩니다.

또한 Council의 2/3 이상의 찬성으로 referendum을 취소할 수 있습니다.

제안은 Root에 의해서 블랙리스트에 오를 수 있습니다. 블랙리스트와 관련된 제안은 즉시 취소됩니다. 또한 블랙리스트에 오른 제안의 해시는 대기열에 다시 등록할 수 없습니다.

![Untitled](https://i.imgur.com/NJWOOTV.png)

모든 Holder는 Council 후보에 자유롭게 등록할 수 있습니다. Council 선거는 Validator를 선정하는 로직과 동일한 로직으로 처리됩니다. Council의 임기는 1주일간 지속됩니다.

매 임기 말에 선거 알고리즘이 실행되고 모든 Voter들은 새로운 councillors를 투표합니다.

**Technical Committee**

Technical Committee는 악의적인 referendum으로 부터 보호하고 버그 수정을 구현하고 잘못된 업데이트를 되돌리거나 새로운 기능을 추가하는 역할을 합니다.

Technical Committee의 멤버는 Council에서 과반수의 투표로 구성원을 더하거나 제할 수 있습니다.

Technical Committee는 Democracy Pallet를 사용하여 제안을 빠르게 추적할 수 있는 권한이 있습니다.

### **Governance V2**

v2에서 변경사항은 아래와 같습니다.

- Council의 역할을 democracy vote를 통해 토큰 홀더들에게 이전
- Council 해산
- 사용자가 투표를 위임할 수 있는 더 많은 방법을 허용

![Untitled](https://i.imgur.com/KylI1Vu.png)

v2에서 누구나 언제든지 referendum를 원하는만큼 여러번 수행할 수 있습니다. v2에서는 `Origins` 및 `Tracks` 라는 새로운 개념이 도입되었습니다.

Origin은 각 referendum class와 연결되고 각 클래스는 Track과 연결됩니다. Track은 해당 제안의 lifecycle을 표시하며 다른 클래스의 Track과 독립적입니다.

v2에서 referendum이 생성되면 커뮤니티에서 즉시 투표할 수 있습니다. 그러나 referendum이 종료할 수 있는 상태에 있지 않을 떄 약식으로 제정됩니다. 여러 기준을 충족하기 전까지 미정 상태로 남아 있습니다.

- lead-in period - 결정을 하기 전 경과해야 하는 시간입니다.
- 결정을 위한 공간이 있어야합니다.
- Decision Deposit이 지불되어야 합니다.

v2에서는 제안이 승인될 수 있는 28일이 주어집니다. 이 기간 동안 승인되지 않으면 제안은 자동으로 거부됩니다.

v2에서는 adaptive quorum biasing이 사라집니다.

Approval은 승인 투표 가중치 / 총 투표 가중치 로 정의됩니다.

Support는 승인 투표 수 / 시스템 총 투표수 로 정의됩니다.

확인 기간동안 최소한의 기준을 충족하여야 합니다. 다른 Track는 다른 확인 기간 및 다른 Approval, Support 기준을 가지고 있습니다.

v2에서는 **Cancelation** 이라고 하는 특수한 액션이 있습니다. 이는 이미 투표중인 제안에 대해서 즉시 거부할 수 있습니다.

v2에서는 Technical Committee를 대체하기위해 Polkadot Fellowship가 도입되었습니다.

Polkadot Fellowship은 기존 Technical Committee와 달리 훨씬 광범위하고 진입장벽이 낮게 설계되었습니다. Polkadot Fellowship의 후보가 되기 위해 소액의 토큰을 예치할 수 있습니다. Polkadot Fellowship 구성원은 Polkadot Fellowship의 제안에 대해 투표할 수 있습니다. Polkadot Fellowship내에 사용되는 제안에 대한 메커니즘은 referendum에 사용되는 매커니즘과 동일합니다.

### Identity

Polkadot은 온체인에 개인정보를 추가할 수 있고 이를 등록기관에 확인을 요청할 수 있는 naming system을 제공합니다.

사용자는 이름, Website 등 사용자 지정 필드를 등록하여 Identity를 설정할 수 있습니다.

사용자는 자신의 정보를 온체인에 저장하기위해 자금을 예치해야 합니다. 이는 항목이 삭제되면 반환됩니다.

사용자는 자신의 정보를 온체인에 등록한뒤 등록기관에 판단을 요청할 수 있습니다. 사용자는 판단에 대해 지불할 용의가 있는 최대 수수료를 설정하고 등록기관은 판단을 해줄 수 있는 최소금액을 지정합니다.

등록기관은 판단을 내릴때 최대 6단계의 신뢰도를 선택할 수 있습니다.

등록기관이 되기 위해서 Democracy에 제안을 제출한 다음 투표를 기다려야 합니다.

### **Nominator**

Nominators는 Relay Chain의 validators를 선택합니다.

validators를 선택하면 validators가 받는 보상을 일부 나눠 받을 수 있습니다.

### **Polkadot Host**

Polkadot의 아키텍쳐는 Runtime과 Host 두 부분으로 나눌 수 있습니다.

Polkadot Runtime은 체인의 코어 로직이 포함되어 있으며 하드포크 없이 업그레이드 할 수 있습니다.

Polkadot Host는 Runtime을 실행하기 위한 환경입니다.

Polkadot Host는

- Libp2p와 같은 네트워크 컴포넌트
- State Storage
- Consensus Engine ( GRANDPA, BABE )
- WASM Interperter and VM
- Low Level Primitives ( Such as Crypto )

로 구성되어 있습니다.

![Untitled](https://i.imgur.com/tpqc6lZ.png)

### **Runtime Upgrades**

Polkadot 온체인에 저장된 로직을 업그레이드 하여 런타임을 업그레이드할 수 있습니다.

### **Transaction Fees**

Relay Chain의 수수료는 3가지 변수에 의해서 지정됩니다.

- Weight Fee
- Length Fee
- Tip

수수료의 일부는 블록 생성자에게 전달되고 나머지는 Treasury로 전달됩니다.

Polkadot의 블록은 Maximum Weight, Maxinmum Length가 지정되어 있습니다. 블록 생성자는 트랜잭션들을 블록에 해당 제한을 초과히지 않도록 채워야 합니다. 블록 생성자는 블록에 최대 75%만 채우며 25%는 체인 운영과 관련된 트랜잭션을 위해 예약되어 있습니다.

Polkadot의 수수료는 3가지로 구성됩니다.

- Base Fee: Runtime에 의해 고정된 매 트랜잭션에 대해 발생하는 수수료
- Length Fee: 트랜잭션의 길이에 따라 부과되는 수수료
- Weight Fee: runtime function의 가중치마다 부과되는 수수료

`fee = base_fee + length_of_transaction_in_bytes * length_fee + weight_fee`

Polkadot은 네트워크가 너무 바쁘면 트랜잭션에 추가 수수료를 부과할 수 있습니다.

`final_fee = fee * fee_multiplier`

Polkadot의 샤드 (Parachain, Parathread) 내에서 발생되는 거래에 대해서는 Relay Chain 수수료가 발생하지 않습니다.

### **Treasury**

Treasury는 block production rewards, transaction fees, slashing, staking inefficiencies 등 자금을 모아둔 버켓입니다.

Treasury에 보관된 자금은 Council에서 승인한 경우 대기기간에 들어가는 지출 제안을 작성하여 사용할 수 있습니다. 이 대기 기간을 지출 기간이라고 하며 거버넌스에 따라 다르지만 현재 24일로 설정되어 있습니다. Treasury는 자금이 부족하지 않은 상태를 유지하면서 대기열에 있는 최대한 많은 제안을 시도하려 합니다.

Treasury가 자금이 고갈 된 상태에서 승인된 제안이 남아있다면 이는 승인된 대기열에 저장되며 다음 지출 기간에 자금 집행이 이루어 집니다.

StakeHolder가 Treasury 에서 지출을 제안하려는 경우 지출의 최소 5%를 예치해야 합니다.

Treasury는 아래의 방법으로 자금을 조달합니다

- Slashing: validator가 어떤 이유로든 Slashing 되었다면 제보자에게 보상과 함께 해당 자금은 Treasury로 보내집니다.
- Transaction Fee: 수수료중 일부는 Treasury에 들어오며 일부는 블록 생성자에게 보상으로 주어집니다.
- Staking inefficiency: 인플레이션은 첫해 10%로 설계되었으며 이상적인 스테이킹 비율은 50%입니다. 즉 모든 토큰의 절반이 스테이킹 되어야 합니다. 이 비율에서 벗어나게 되면 이에 비례하는 인플레이션 비율이 Treasury에 전송됩니다.
- Parathread: Parathread는 블록에 포함되기 위하여 경매에 참여합니다. 입찰금액의 일부는 Validator로 전달되며 나머지는 Treasury에 전송됩니다.

**Creating a Treasury Proposal**

제안자는 스팸 방지로 최소 100 DOT 혹은 요청 금액의 5%를 예치하여야 하며 최대 한도는 500 DOT입니다. 이 금액은 제안이 거부되면 소각되거나 환불됩니다.

제안이 제출된 이후에 사용자는 이를 철회할 수 없습니다.

Bounties Spending proposals은 큐레이터라고 하는 전문가에게 버그 또는 취약점 수정, 전략 개발, 모니터링 등을 목표로 하는 제안입니다.

제안자는 Council에서 통과할 수 있도록 제안을 제출할 수 있으며, 이후 큐레이터를 지정할 수 있으며 제안이 통과되면 큐레이터는 예치금을 일부 예치해야하며 이는 큐레이터가 악의적으로 행동할 경우 처벌이 될 수 있습니다. 바운티 작업을 성공적으로 완료하였다면 예치금과 바운티 보상의 일부를 받을 수 있습니다.

### Validator

Validator는 Collator의 Proof을 검증하고 다른 Validators와 합의에 참여하여 Relay Chain을 보호하는 역할을 수행합니다.

Validators는 Relay chain에 새로운 블록을 추가하는 역할을 합니다.

## Staking

### **Introduction to Staking**

**Nominated Proof-of-Stake (NPoS)[](https://wiki.polkadot.network/docs/learn-staking#nominated-proof-of-stake-npos)**

Polkadot은 Nominated Proof-of-Stake (NPoS)을 구현하였습니다. 모든 유저는 validators 후보가 될 수 있습니다. validators 후보는 Nominator에 공개되며 Nominator는 차례로 최대 16명의 후보를 선택하여 제출할 수 이습니다.

Polkadot에서 사용하는 선거 알고리즘은 Phragmen와 같은 Proportional Justified Representation에 기반을 두고 있습니다.

active set of validators가 활동하는 24시간의 기간을 ear라고 합니다.

각 ear는 6개의 epoch로 나뉘며 각 epoch동안 validators는 특정 슬롯에 블록 생성자로 할당됩니다.

블록을 생성하는 validators는 토큰을 보상을 받을 수 있고 이를 nominator와 나눌 수 있습니다.

validators와 nominators 모두 ear가 끝날 떄 스테이킹 보상을 받을 수 있습니다. 이는 스테이킹 수량에 관계없이 균등하게 지급됩니다.

**Slashing**

Slashing은 validator가 네트워크 상에서 잘못된 행동을 했을때 발생합니다. validator와 그의 nominators 이 같이 Slashed 되며 Staking한 DOT은 모두 손실됩니다. 여기서 손실된 DOT은 Treasury로 귀속됩니다.

**Chilling**

Chilling은 validator이나 nominator가 사임하는 행위입니다. 이는 validator이나 nominator 양쪽 모두 수행할 수 있으며 다음 era부터 적용됩니다.

validator가 Chilling을 수행했을 경우 nominator를 잃지 않습니다.

Polkadot은 현재 297개의 Validator가 있습니다. 상한은 아직 정해져있지 않지만 네트워크 대역폭 등에 부담이 될 수 있어 제한 되어야 합니다.

안정기에는 약 1000개의 Validator가 될 것으로 예측 됩니다.

### **Nomination Pools**

![Untitled](https://i.imgur.com/83yVKkk.png)

**Key Compoments**

- Bonded Pool: actively staked funds의 분배를 추적
- Reward Pool: actively staked funds로 획득한 보상을 추적
- Unbonding Sub Pools: 각각 다른 phases의 lifecycle에서의 풀의 모음
- Members: Pool에 Nominate된 계정
- Point: Pool에 대한 각 멤버들의 비중

**Pool Member Lifecycle[](https://wiki.polkadot.network/docs/learn-nomination-pools#pool-member-lifecycle)**

Join a pool

Pool에 들어가기 위한 최소 예치금은 1 DOT입니다. 사용자는 한번에 하나의 Pool에만 속할 수 있습니다.

Claim rewards

사용자는 이전 보상 청구한 시점부터 누적된 보상을 청구할 수 있습니다.

Unbond and withdraw funds

Pool에 가입한 후 언제든지 예치금을 빼는것으로 Pool에서 탈퇴할 수 있습니다.

---
