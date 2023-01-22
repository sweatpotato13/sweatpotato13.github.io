---
layout: post
title: "Polkadot 개념 정리 advanced"
date: "2023-01-22 13:03:44 +0900"
tags: [blockchain]
---

본 문서는 Polkadot wiki를 읽고 나름대로 정리한 문서입니다. Polkadot에 대한 기본 이해가 부족한 상태에서 읽고 작성한 문서이기 때문에 본문에 오류가 있을 수 있습니다. 실제 문서는 [Learn](https://wiki.polkadot.network/docs/learn-account-advanced) 부터 시작되는 Advanced 부분을 참고하세요.

# Accounts

## **Address Format[](https://wiki.polkadot.network/docs/learn-account-advanced#address-format)**

Substrate 기반 체인에서는 `SS58` 기반의 address format을 사용하고 있습니다.

`SS58` 은 Bitcoin의 Base-58-check를 일부 수정한 버전입니다. 이 형식에는 특정 네트워크에 접속했다는 내용을 포함한 prefix가 포함되어 있습니다.

[ss58 Formats | polkadot{.js}](https://polkadot.js.org/docs/keyring/start/ss58/)

예를 들어

- Polkadot 주소는 항상 숫자 1로 시작합니다.
- Kusama 주소는 항상 영어 대문자 C, D, F, G, H, J 로 시작합니다.
- 일반적인 Substrate 주소는 항상 숫자 5로 시작합니다.

이러한 주소의 형태는 PublicKey-PrivateKey 쌍을 다르게 표현한 것입니다.

## **Derivation Paths[](https://wiki.polkadot.network/docs/learn-account-advanced#derivation-paths)**

동일한 Seed를 사용하여 네트워크에서 여러 계정을 만들고 관리하려는 경우 derivation path를 사용할 수 있습니다. 이는 니모닉 문구를 사용하여 생성된 루트 계정의 하위 계정으로 생각할 수 있습니다.

Polkadot 체인에서 사용할 계정을 만드려는 경우 니모닉 문구 뒤에 `//`를 사용하여 **hard key**
 child account를 만들 수 있습니다.

```
'caution juice atom organ advance problem want pledge someone senior holiday very//0'
```

혹은 `/` 를 붙여 **soft key** child account를 만들 수 있습니다.

```
'caution juice atom organ advance problem want pledge someone senior holiday very/0'
```

derivation path에 문자나 숫자를 사용할 수 있습니다. 위 예시에는 `//0` 이나 `/0` 등 숫자를 사용하였지만 `//polkadot` 이나 `//cute/wisp` 등 문자를 사용하거나 여러 path를 결합할 수 있습니다. derivation path를 사용하는 경우 니모닉 문구와 path를 모두 알고있어야 합니다.

니모닉 문구 뒤에 `///`를 붙여 password key account를 파생할 수 있습니다.

```
'caution juice atom organ advance problem want pledge someone senior holiday very///0'
```

이러한 파생 방식에서는 니모닉 문구가 유출되는 경우 초기 암호 없이 계정을 파생시킬 수 없습니다.

soft, hard derivated account 에서는 니모닉 구문과 path만 알고있으면 계정에 접근할 수 있습니다.

password key account에서는 다른 파생 방법과는 달리 path당 하나의 암호만을 지정해야 합니다.

### **Soft vs. Hard Derivation[](https://wiki.polkadot.network/docs/learn-account-advanced#soft-vs-hard-derivation)**

soft derivation을 통해서 파생된 계정의 개인키를 알고 있는 경우 이를 가지고 역추적 하는 경우 초기 계정의 개인키를 알아낼 수 있습니다. 동일한 Seed에서 생성된 다른 계정이 해당 시드에 연결되어 있는지도 확인이 가능합니다.

hard derivation의 경우 파생된 계정의 개인키를 알고있다 하더라도 초기 계정의 개인키를 알아내는것은 불가능합니다.

# **Advanced Staking Concepts**

## **Staking Proxies[](https://wiki.polkadot.network/docs/learn-staking-advanced#staking-proxies)**

### **Stash is also Controller[](https://wiki.polkadot.network/docs/learn-staking-advanced#stash-is-also-controller)**

![Untitled](https://i.imgur.com/dvwsvSw.png)

Polkadot에서는 Proxy 계정을 별도로 설정할 수 있습니다. 그 중 스테이킹 관련 액션만 할 수 있도록 권한을 주어 Proxy를 설정할 수도 있습니다.

Staking Proxy가 있는 경우 Stash도 Controller의 역할을 수행할 수 있습니다. Staking Proxy를 변경하기 위해서는 Stash의 서명이 필요합니다.

이 경우에서 Staking Proxy는 모든 스테이킹 관련 액션을 호출할 수 있지만 Balance 관련 Pallet은 호출할 수 없습니다. 이는 Staking Proxy는 Balance를 transfer할 수 없음을 의미합니다.

### **Stash is not Controller[](https://wiki.polkadot.network/docs/learn-staking-advanced#stash-is-not-controller)**

![Untitled](https://i.imgur.com/Xcy73T3.png)

만약 Stash와 Controller가 다른 계정이라면 Staking Proxy는 추가 스테이킹 및 Controller 변경을 위해 사용됩니다. 따라서 Staking Proxy는 많이 사용되지 않습니다.

이런 구조에서는 스테이킹 작업을 완전히 제어하려면 Staking Proxy, Controller 2개의 계정에 모두 접근할 수 있어야 합니다.

## **Bags List (voterList)**

![Untitled](https://i.imgur.com/yyw4gie.png)

Polkadot NPoS에서 Nomination 후보들은 bags-list 라고 하는 리스트에 포함됩니다.

Bags List 는 Bags와 Node 2가지의 요소로 구성되어 있습니다. Bags 에는 일정 범위의 스테이킹 된 잔액이 있는 노드가 포함되어 있습니다.

Bags는 Nomination 후보들이 자동으로 투입되지만 지분이 아닌 시간순으로 정렬됩니다.

![Untitled](https://i.imgur.com/BDDAya4.png)

각 Bag의 아래에서 위로 올라가기 위해서 `voterList` Pallet에 있는 `putInFrontOf` 액션을 수행할 수 있다.

![Untitled](https://i.imgur.com/InLHLj0.png)

위 그림에서 Bag에 아래에 있던 19 DOT 노드에 2개가 추가된다면 1번째 Bag의 맨 위로 이동할 수 있습니다.

만약 스테이킹 보상을 Stash보상으로 보내고 자동으로 스테이킹하기로 결정했다면 Bag내 위치는 자동으로 변하지 않습니다. 이는 Slashing이 발생했을때도 동일하게 적용됩니다. 예를들어 Nominator가 해임되었을 경우에 Bag에서 위치는 변하지 않습니다. 이로인해 Bag에 위치한 노드가 올바른 Bag으로 이동해야 하는 경우가 있을 수 있습니다. 이런 문제를 해결하기 위해 온체인 계정은 `voterList` Pallet에 `rebag` 을 호출하여 Bag에 속하지 않은 노드 위치를 업데이트하여 올바른 Bag에 위치시키게 끔 할 수 있습니다.

즉 Token Bonding / Unbonding 같은 액션은 자동으로 `rebag` 을 실행하지만 스테이킹 보상이나 Slashing 같은 액션은 자동으로 `rebag` 을 실행하지 않습니다.

Bag List는 체인의 런타임 스토리지에 따라서 무제한의 노드를 포함할 수 있습니다.

## **Rewards Distribution[](https://wiki.polkadot.network/docs/learn-staking-advanced#rewards-distribution)**

Validators 보상은 Staking 양과 관계 없이 동일하게 지급됩니다. 이는 소수의 Validators에게 권한이 집중되는것을 막을 수 있습니다. 장기적으로 보면 Validators는 비슷한 수준의 지분을 가지게 될 것이며 이는 비중이 낮은 Validators를 지원한 Nominator은 더 많은 위험을 감수한 만큼 더 많은 보상을 받을 수 있게 됩니다.

Nominator에게 보상을 분배하기 전에 Validator는 Nominator와 분배되지 않는 보상인 커미션을 생성할 수 있습니다. 이는 블록 보상의 백분율 입니다.(일정 퍼센트 지정 가능) 커미션이 공제된 후 나머지 금액은 Staking에 따라 Nominator와 Validator에게 분배됩니다.

Validator가 가질 수 있는 Nominator수 에는 제한이 없지만 보상을 받을 수 있는 Nominator수는 제한이 있습니다. 이는 현재 256개로 되어 있지만 런타임 업그레이드를 통해 수정될 수 있습니다. Nominator들 중 Staking에 따라 상위 256명 까지만 보상을 받습니다.

## **Slashing[](https://wiki.polkadot.network/docs/learn-staking-advanced#slashing)**

### **Unresponsiveness[](https://wiki.polkadot.network/docs/learn-staking-advanced#unresponsiveness)**

모든 세션에 대해서 validators는 heartbeat를 보내 온라인임을 나타내야 합니다. validators가 epoch동안 블록을 생성하지 않고 heartbeat가 실패한다면 응답하지 않는것으로 간주합니다. 이런 행동이 반복되면 Slashing 처리가 될 수 있습니다.

### **Equivocation**

GRANDPA: validators가 다른 체인의 동일한 2라운드에서 2개 이상의 vote에 서명한 경우

BABE: validators가 Relay chain의 동일 slot에서 2개 이상의 블록을 생성한 경우

이 경우 Slashing이 될 수 있습니다.

### Slashing Across Eras

nominator는 여러 validator에 지원을 할 수 있으며 이들로 인해 Slashing 될 수 있습니다. Slashing 될 떄 일정 기간동안 받을 수 있는 Slashing중 가장 큰 건에 대해서만 Slashing 됩니다

## **Simple Payouts[](https://wiki.polkadot.network/docs/learn-staking-advanced#simple-payouts)**

Polkadot에서 이전 era의 보상을 받기 위해서 트랜잭션을 제출하여야 합니다. 이는 모든 Staker 들이 단일 블록에서 보상을 받는것이 아닌 다른 시간에 트랜잭션을 날리게 하여 자연스럽게 보상을 분배하기 위함힙니다.

Polkadot은 최근 84 era를 저장하고 있습니다. 보상은 이 84 era가 지나면 청구할 수 없습니다. 즉 모든 보상은 이 기간 이내에 청구하여야 합니다. validator가 자신의 stash를 죽이면 보상을 더이상 수령할 수 없습니다. 이를 수행하기 전에 먼저 validating을 중지한 다음 stash에 있는 자금을 언스테이킹 해야 합니다.

거래 수수료를 지불할 의향이 있는 누구나 Vaildator에 대한 보상 트랜잭션을 날릴 수 있습니다.

## **Inflation**

DOT은 인플레이션이 발생합니다. DOT은 발행 상한이 존재하지 않습니다. 인플레이션은 매년 10%입니다. Vaildator 보상 이외의 자금은 Treasury로 이동합니다.

네트워크가 유지하고자 하는 이상적인 스테이킹 비율이 있습니다. 시스템은 이상적인 스테이킹 비율을 유지하고자 합니다. 이상적인 스테이킹 비율은 Parachain의 수에 따라 다릅니다.

스테이킹된 토큰 양이 이상적인 비율보다 낮아지면 Nominator에 대한 스테이킹 보상이 증가하여 더 많은 스테이킹을 유도하고 반대로 이상적인 비율을 초과한다면 스테이킹 보상이 떨어집니다.

![Untitled](https://i.imgur.com/1hMVhtK.png)

# **Availability and Validity**

## **Phases of the AnV protocol[](https://wiki.polkadot.network/docs/learn-availability#phases-of-the-anv-protocol)**

Availability 과 Vaildity 프로토콜에는 5단계가 있습니다.

1. Parachain phase.
2. Relay Chain submission phase.
3. Availability and unavailability subprotocols.
4. Secondary GRANDPA approval validity checks.
5. Invocation of a Byzantine fault tolerant *finality gadget* to cement the chain.

### **Parachain phase[](https://wiki.polkadot.network/docs/learn-availability#parachain-phase)**

AnV의 Parachain phase는 collator가 Validator에게 블록 후보를 제안하는 단계입니다.

### **Relay Chain submission phase[](https://wiki.polkadot.network/docs/learn-availability#relay-chain-submission-phase)**

validators는 블록 후보를 확인합니다. 확인이 완료되었으면 validators는 후보 블록을 다른 validators에게 전달합니다. 검증에 실패하였다면 그 후보 블록은 즉시 거부됩니다.

validators는 각각의 Parachain에 대한 할당을 결정하고 후보 블록에 대한 승인을 결정하고 무효 후보 블록에 대해서 처리를 해야합니다. 각각의 모든 validator가 모든 parachain 후보를 검증할 수 없기 때문에 parachain 후보를 선택하기 위해서 정직한 validator가 선택되도록 해야합니다.

정직한 validator가 승인된 유효하지 않은 블록을 발견한다면 악의적인 validator에게 처벌을 유발하는 dispute를 제시해야 합니다.

validators의 절반이상이 블록 후보에 대해서 승인한다면 candidate receipt를 발행합니다. 이는 결국 Relay Chain State에 포함될 것입니다. 이는 아래와 같은 내용을 포함하고 있습니다.

- The parachain ID.
- The collator's ID and signature.
- A hash of the parent block's candidate receipt.
- A Merkle root of the block's erasure-coded pieces.
- A Merkle root of any outgoing messages.
- A hash of the block.
- The state root of the parachain before block execution.
- The state root of the parachain after block execution.

### **Availability and unavailability subprotocols[](https://wiki.polkadot.network/docs/learn-availability#availability-and-unavailability-subprotocols)**

validators는 erasure coded pieces를 네트워크에 퍼뜨립니다. 적어도 validators중 1/3 + 1은 자신의 code word를 소유하고 있음을 알려야 합니다.

## **Erasure Codes**

Erasure coding은 메시지를 더 긴 코드로 변환하여 코드의 하위집합에서 원본 메시지를 복구할 수 있도록 합니다

Code는 복구할 수 있도록 원문 메시지에 패딩값을 추가한 메시지 입니다. Polkadot의 availability scheme에서 사용되는 Erasure code 유형은 Reed-Solomon codes로 이는 많은곳에서 사용되고 있습니다.

Polkadot에서 erasure codes는 모든 Validator가 모든 Parachain에 대해서 항상 검증하지 않아도 Parachain State를 시스템에 서 사용할 수 있도록 유지하는 역할을 합니다. 대신 validators는 더 작은 code 조각을 공유하고 validators중 1/3 +1 이 이를 제공할 수 있다 전체 데이터를 구성할 수 있게 됩니다.

# **Cross-Consensus Message Format (XCM)**

**[](https://wiki.polkadot.network/docs/learn-xcm#a-format-not-a-protocol)**

XCM(Cross-Consensus Message Format)은 consensus systems간 통신을 목표로 합니다.

Polkadot은 상호운용성을 지향하고 있으며 XCM은 이를 구현하기 위한 수단입니다.

아래의 수단들을 가능하게 하는 VM과 함께 제공됩니다.

1. **Asynchronous**: XCM 메시지는 보낸 사람이 메시지 전송을 완료했을때 차단할 것이라 가정하지 않습니다.
2. **Absolute**: XCM 메시지는 순서대로 전달됨을 보장합니다.
3. **Asymmetric**: XCM 메시지에는 보낸 사람에게 메시지를 정상적으로 수신했다는 알람을 줄 수 없습니다. 이 같은 행동을 구현하려면 별도로 메시지를 보내주어야 합니다.
4. **Agnostic**: XCM은 메시지가 전달되는 시스템의 Consensus에 상관없이 동작합니다.

## **A Format, Not a Protocol**

XCM은 실제 시스템간 메시지를 보낼 수 없습니다.

UDP와 비슷하게 응답으로 설계된 XCM 메시지가 없다면 XCM은 자동으로 응답을 보내주지 않습니다.

## **Asset Teleportation[](https://wiki.polkadot.network/docs/learn-xcm#asset-teleportation)**

![Untitled](https://i.imgur.com/1qimgwQ.png)

1. InitiateTeleport

   Source는 전송하려는 계정에서 자산을 빼냅니다.

2. ReceiveTeleportedAsset

   Source는 `ReceiveTeleportedAsset` 라는 XCM 메시지를 자산의 양과 수신 계정을 포함하여 생성합니다. 이 메시지를 Destination으로 보내면 새 자산이 Destination에 투입됩니다.

3. DepositAsset

   Destination은 수신 계정에 자산을 예치합니다.

### **Reserve Asset Transfer[](https://wiki.polkadot.network/docs/learn-xcm#reserve-asset-transfer)**

Consensus System에 자산을 전송할 수 있는 계층이 없는경우 신뢰할 수 있는 다른 엔티티를 선택할 수 있습니다.

![Untitled](https://i.imgur.com/S7sfktL.png)

1. InitiateReserveWithdraw

   Source는 송금 계정에서 이체할 자산을 모아서 소각하고 이를 기록합니다

2. WithdrawAsset

   Source는 예약을 위해 `WithdrawAsset` 명령을 보내고 sovereign account에서 소각한 양만큼을 예약합니다.

3. DepositReserveAsset

   Reserve는 이전에 이체된 양만큼 Destination의 sovereign account에 예치를 예약합니다.

4. ReserveAssetReposited

   Reserve는 `ReserveAssetDeposited` 명령을 정해진 자산 만큼을 포함해서 보냅니다. Destination 은 이를 받아서 처리합니다. 이는 Destination 쪽에서 신규 발행이 될 것 입니다.

5. DepositAsset

   Destinationdms 신규 발행된 양을 수금 계정에서 수신합니다.

### **XCM Tech Stack[](https://wiki.polkadot.network/docs/learn-xcm#xcm-tech-stack)**

![Untitled](https://i.imgur.com/nXVhm73.png)

## **XCVM (Cross-Consensus Virtual Machine)[](https://wiki.polkadot.network/docs/learn-xcm#xcvm-cross-consensus-virtual-machine)**

XCM의 핵심에는 XCVM이 존재합니다. XCM의 메시지는 XCVM의 프로그램 입니다. XCVM은 state machine이며 state는 레지스터에서 추적됩니다.

## **Cross-Consensus Protocols (XCMP, VMP, HRMP)[](https://wiki.polkadot.network/docs/learn-xcm#cross-consensus-protocols-xcmp-vmp-hrmp)**

XCM 형식이 설정되면 메시지의 프로토콜에 대한 공통 패턴이 필요합니다. Polkadot은 Parachain간에 XCM 메시지를 작용하기위한 2가지 메시지 전달 프로토콜을 사용합니다.

### **XCMP (Cross-Chain Message Passing)[](https://wiki.polkadot.network/docs/learn-xcm#xcmp-cross-chain-message-passing)**

<aside>
🚧 XCMP는 현재 개발중이며 Cross-Chain Message는 당분간 HRMP를 사용합니다.

</aside>

REST - RESTful 관계처럼 XCM - XCMP 관계가 이루어집니다.

Cross-chain Message Passing은 두 Parachain간 안전한 메시지 전송을 가능하게 합니다.

이는 2가지 방법이 있습니다

- Direct: 메시지가 Parachain간 다이렉트로 전송됨
- Relayed: 메시지가 Relay Chain을 통해서 전송됨

### **VMP (Vertical Message Passing)[](https://wiki.polkadot.network/docs/learn-xcm#vmp-vertical-message-passing)**

Relay Chain과 Parachain간 메시지 전송에서는 두 경우 모두 Relay chain에 메시지가 존재하게 됩니다.

- UMP (Upward Message Passing)[](https://wiki.polkadot.network/docs/learn-xcm#ump-upward-message-passing)
  Parachain에서 Relay Chain으로 메시지 전송
- DMP (Downward Message Passing)[](https://wiki.polkadot.network/docs/learn-xcm#dmp-downward-message-passing)
  Relay Chain에서 Parachain으로 메시지 전송

### **HRMP (XCMP-Lite)[](https://wiki.polkadot.network/docs/learn-xcm#hrmp-xcmp-lite)**

XCMP가 구현되는동안 HRMP라고 하는 임시 프로토콜을 사용합니다.

XCMP와 동일한 인터페이스와 기능을 가지고 있지만 Relay Chain Storage에 모든 메시지를 저장하기 떄문에 더 많은 리소스를 요구합니다.

# **Cryptography**

## **Hashing Algorithm[](https://wiki.polkadot.network/docs/learn-cryptography#hashing-algorithm)**

Polkadot은 Hash 알고리즘으로 Blake2b를 사용하고 있습니다.

## **Keypairs and Signing[](https://wiki.polkadot.network/docs/learn-cryptography#keypairs-and-signing)**

Polkadot은 Key derivation 및 signing 알고리즘으로 x25519(sr25519)를 사용하고 있습니다.

Sr25519는 Ed25519와 동일한 커브인 Curve25519를 기반으로 하고 있습니다. 하지만 EdDSA대신 Schnorr 서명을 사용하고 있습니다.

## **Keys[](https://wiki.polkadot.network/docs/learn-cryptography#keys)**

### **Account Keys[](https://wiki.polkadot.network/docs/learn-cryptography#account-keys)**

계정 키는 아래 중 하나일 수 있습니다.

- Ed25519
- sr25519
- secp256k1

### **"Controller" and "Stash" Keys[](https://wiki.polkadot.network/docs/learn-cryptography#controller-and-stash-keys)**

Controller 키는 사용자가 직접 제어할 수 있는 키이며 트랜잭션을 제출하는데 사용됩니다. 이는 Controller 키를 이용하여 Validating 혹은 Nomination을 시작하거나 정지함을 의미합니다. Controller 키는 수수료를 지불하기 위해 약간의 DOT을 보유해야 하며, 대량의 금액을 보유하고 있기에 적합하지 않습니다.

Stash 키는 콜드 월렛이 되는 키 입니다. 인터넷에 노출되거나 트랜잭션을 제출하는데 사용되지 않습니다. Stash 키는 오프라인 상태로 유지되므로 자산이 특정 Controller에 Bond 되도록 설정 해야합니다.

### **Session Keys[](https://wiki.polkadot.network/docs/learn-cryptography#session-keys)**

Session 키는 Validator가 네트워크 작업을 수행하기 위해 온라인 상태로 유지해야하는 키 입니다. Session 키는 자산을 통제하기 위한것이 아니며 의도된 목적으로만 사용하여야 합니다. Controller는 Session 공개키에 서명하여 인증서를 만들고 외부에 이 인증서를 전파하면 됩니다.

Polkadot은 6개의 Session 키를 사용하고 있습니다.

- Authority Discovery: sr25519
- GRANDPA: ed25519
- BABE: sr25519
- I'm Online: sr25519
- Parachain Assignment: sr25519
- Parachain Validator: ed25519

# **NPoS Election Algorithms**

Validator의 보상이 각 era 마다 거의 균등하게 지급되기 때문에 각 Validator의 지분이 균일하게 분산되는것이 중요합니다. NPoS 알고리즘은 아래 값을 최적화 하려고 시도합니다.

- 총 스테이킹 양을 극대화
- 최저 스테이킹 Validator의 값을 극대화
- 스테이킹의 Variance를 최소화

## **What is the sequential Phragmén method?[](https://wiki.polkadot.network/docs/learn-phragmen#what-is-the-sequential-phragm%C3%A9n-method)**

### **Validator Elections[](https://wiki.polkadot.network/docs/learn-phragmen#validator-elections)**

sequential Phragmén은 NPoS에서 Validator을 선출하는데 사용됩니다. 이는 각 투표시마다 Validator의 비중을 균등하게 합니다.

### **Council Elections[](https://wiki.polkadot.network/docs/learn-phragmen#council-elections)**

sequential Phragmén은 Council 선출에도 사용됩니다.

## **Understanding Phragmén[](https://wiki.polkadot.network/docs/learn-phragmen#understanding-phragm%C3%A9n)**

## **Basic Phragmén[](https://wiki.polkadot.network/docs/learn-phragmen#basic-phragm%C3%A9n)**

### **Rationale[](https://wiki.polkadot.network/docs/learn-phragmen#rationale)**

기본적인 Phragmén을 알아야 합니다.

후보자그룹, 경쟁자그룹, 투표자그룹이 있어야 합니다. 또한 모든 투표자는 동일한 투표권이 있다고 가정합니다.

### **Algorithm[](https://wiki.polkadot.network/docs/learn-phragmen#algorithm-1)**

기본적인 Phragmén은 다음 규칙을 반복하면서 한 자리씩 선택합니다.

1. 유권자는 본인이 지지하는 후보자에게 투표합니다.
2. 각 투표마다 초기 로드를 0으로 설정합니다.
3. 다음 자리를 차지할 후보는 투표의 평균 로드가 가장 작은 후보입니다.
4. 당선 후보를 선택한 n개의 투표용지에 로드를 1/n을 추가합니다.
5. 이번 라운드의 승자를 지지한 모든 투표용지의 평균을 내어 균등하게 합니다.
6. 선택해야 할 자리가 더 있으면 3번으로 돌아가 이를 반복합니다.

## **Weighted Phragmén[](https://wiki.polkadot.network/docs/learn-phragmen#weighted-phragm%C3%A9n)**

### **Rationale[](https://wiki.polkadot.network/docs/learn-phragmen#rationale-1)**

Polkadot 에서 Validator, Council 투표는 모두 유권자가 보유한 토큰 지분에 따라 가중치가 부여됩니다.

### **Algorithm[](https://wiki.polkadot.network/docs/learn-phragmen#algorithm-1)**

1. 후보자는 한 라운드당 한명씩 선출되어 successful candidate에 추가됩니다.
2. 후보자가 선출이 되면 가중치 맵핑이 구축되어 각 Nominator가 선택한 Validator의 가중치를 정의합니다.

3. 모든 투표자는 자신의 스테이킹 금액, Validator의 목록을 작성합니다.
4. 유권자 → 후보자의 초기 edge-weighted 그래프를 생성합니다. 여기서 edge 가중치는 유권자가 보유하고 있는 투표의 가중치입니다. 후보에 대한 모든 잠재적 가중치의 합을 `approval_stake`라고 합니다.
5. 후보 선출을 시작합니다. 선출되지 않은 모든 후보는 `1/approval_stake` 를 획득합니다.
6. 각 유권자에 대해 유권자의 자산에 로드율을 곱한다음 후보의 `approval_stake` 로 나누어 각 후보의 점수를 업데이트합니다.

   `(voter_budget * voter_load / candidate_approval_stake)`.

7. 점수가 가장 낮은 후보를 결정하고 그 후보를 선출합니다.
8. 선출된 후보에 연결되는 각 edge의 가중치는 후보 점수에서 유권자의 가중치를 뺀 값으로 설정된 edge 가중치로 업데이트 하고 유권자의 가중치는 후보 점수로 설정됩니다.
9. 선출할 후보가 더 있으면 3단계 / 아니면 8단계로 이동합니다.
10. 이제 stake는 적어도 한명의 선출된 후보를 지지한 Nominator에게 분배됩니다. 각 후보에 대한 지지 지분은 유권자의 예싼에 edge 부하를 곱한다음 후보 부하로 나누어 계산합니다. (`voter_budget * edge_load / candidate_load`).

# Randomness

Polkadot은 VRF를 사용합니다.

## VRF

VRF는 제출자가 난수를 생성했다는 진위 증명과 함께 일부 입력을 받아 난수를 생성하는 연산입니다. 난수 생성이 유효한지 확인하기 위해 모든 도전자가 증명을 확인할 수 있습니다.

슬롯은 길이가 6초인 시간 단위입니다. 각 슬롯에는 블록이 포함될 수도 있고 아닐 수도 있습니다. 슬롯은 epoch를 구성합니다. Polkadot에서는 2400개의 슬롯이 하나의 epoch를 만들고 4시간동안 Epoch를 만듭니다.

모든 슬롯에서 각 Validator는 주사위를 굴리고 그 아래를 입력으로 하는 VRF를 실행합니다.

- 주사위를 굴릴 때 사용한 Secret Key
- epoch randomness value
- slot number

으로 `RESULT` 와 `PROOF` 가 출력됩니다. 그러나음 프로토콜 구현에서 정의된 임계값과 비교 합니다. 값이 임계값보다 작으면 주사위를 굴린 Validator가 블록 생성 후보가 됩니다.

그 다음 Validator는 블록 생성을 시도하고 이전에 얻은 `RESULT`, `PROOF` 를 네트워크에 제출합니다. VRF에서 모든 Validator는 스스로 주사위를 굴리고 임계값과 비교하여 임계값 미만인경우 블록을 생성합니다.

![Untitled](https://i.imgur.com/SY23RYS.png)

## RANDAO

온 체인에서 Randomness를 얻는 다른 방법은 이더리움의 RANDAO 입니다.

RANDAO는 각 Validator가 seed에 대해서 해시를 수천번 계산하여 획득합니다.

Validator는 마지막 해시값을 가져다가 난수로 사용합니다.

# SPREE

Shared Protected Runtime Execution Enclaves (SPREE) 는 “Trust wormhole” 이라고 불리는 Substrate의 런타임 모듈과 유사한 logic fragments입니다.

하지만 이는 Polkadot Relay Chain에 상주하여 Parachain에 의해 선택될 수 있습ㄴ디ㅏ.

- Parachain은 스마트 컨트랙트와 같이 특별한 runtime logic fragments을 선택할 수 있습니다.
- 이러한 fragments 는 자기 자신의 스토리지와 XCM endpoint를 가지고 있습니다
- Parachain의 모든 인스턴스는 동일한 Logic을 갖습니다.
- 이는 Parachain Login과 함께 실행됩니다.
- 저장소는 Parachain logic으로 변경될 수 없습니다. 메시지는 Parachain에 의해 변경될 수 없습니다.

## Origin

처음에는 SmartProtocol 이라고 하는 아이디어로 시작 하였습니다. 이 개념의 핵심은 XCMP가 신뢰 없이 Parachain에서 실행되었음을 증명하기 어려움을 해결하고자 각 Parachain과의 인터페이스를 통해서만 변경할 수 있는 인스턴스당 자체 저장소가 있는 Relay Chain에 SmartProtocol을 설치하는 것 이었습니다.

## **What is a SPREE module?[](https://wiki.polkadot.network/docs/learn-spree#what-is-a-spree-module)**

SPREE 모듈은 커버넌스 메커니즘이나 Parachain을 통해 Polkadot에 업로드되는 논리 조각입니다. (정확하게는 blobs fo WebAssembly Code) Blob이 Polkadot에 업로드 되면 다른 모든 Parachain 이 로직을 적용하도록 결정할 수 있습니다. SPREE 모듈은 Parachain과 독립적인 자체 스토리지를 가지고 있지만 Parachain과의 인터페이스를 통해서 호출할 수 있습니다.

SPREE 모듈은 대상이 되는 Parachain에서 실행될 코드를 보장하기 때문에 XCMP 아키텍쳐에서 중요합니다. XCMP는 메시지 전달을 보장하지만 어떤 코드가 실행되는지, 즉 받는 쪽에서 어떻게 이를 이해하는지 보장하지 않습니다.

## Why

XCMP에서 Parachain을 통해 메시지를 보내는것은 메시지가 전달 되는것을 보장할 뿐 실행되는 코드나 받는 Parachain에서 메시지를 해석하는 방법을 지정하지 않습니다.

SPREE는 SPREE 모듈과 Parachain간 동일한 logic이 공유되는데 도움이 되도록 합니다.

![Untitled](https://i.imgur.com/qHHZnxf.png)

# **Staking Miner**

<aside>
💡 The staking-miner code is experimental and it is still in the development phase. Use is at your own discretion, as there is a risk of losing some funds.

</aside>

Polkadot 에서 각 era가 끝날때 NPoS를 사용하여 Nominator의 투표에 따라 새로운 Validator 세트를 선출해야 합니다. 이 솔루션을 계산하기 위해서 mining 이라는 용어를 사용합니다.

Validator는 off-chain worker를 사용하여 결과를 계산하고 트랜잭션을 제출합니다. 이런 역할도 off-chain의 다른 프로그램에서 작업을 위임할 수 있습니다.

Staking miner들은 해당 Validator set에 대하여 지분 분배 및 솔루션이 얼마나 최적인지를 나타내는 점수로 구성된 선거 솔루션을 생산하기 위해 서로 경쟁합니다.

Staking miner들은 주어진 스테이킹 알고리즘을 실행하여 생성한 결과를 Relay Chain에 트랜잭션으로 전송합니다. 여기서 수수료를 가장 적게 사용하는 솔루션이 최상의 솔루션으로 책정되어 보상이 주어집니다.

# WebAssembly

Polkadot 및 Substrate에서는 WASM을 사용함으로써 하드포크 없이 런타임 로직을 업그레이드 할 수 있습니다.

---
