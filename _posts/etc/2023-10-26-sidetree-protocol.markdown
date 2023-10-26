---
layout: post
title: "DID Sidetree 프로토콜"
date: "2023-10-26 08:05:44 +0900"
tags: [etc]
---

## Overview

Sidetree 프로토콜은 탈중앙화 시스템 위에서 DID를 구현할 수 있게 해주는 Layer 2 프로토콜 입니다. 

현재 여러 블록체인 위에서 동작하는 많은 DID 시스템이 있지만, 트랜잭션 수수료 및 처리량에 대한 한계가 있기 때문에 그러한 문제점을 해소할 수 있는 방안으로 제시된 프로토콜 입니다.

## Network Topology

![Network Topology](https://i.imgur.com/hTwIACM.png)

Sidetree의 네트워크는 크게 

- Content Addressable Storage (CAS) (eg. IPFS)
- Sidetree Node
- Decentralized Ledger System (eg. Bitcoin, Ethereum)

의 3가지로 구분될 수 있습니다.

![Sidetree Node](https://i.imgur.com/5YY10qn.png)

Sidetree Node는 사용자로 부터 특정 요청을 받아 이를 처리합니다. 사용자로부터 받을 수 있는 요청은 DID resolve, 혹은 Create, Update, Recover, Deactivate 등의 동작이 될 수 있습니다. 

DID State에 대한 변경이 필요하다면 변경 사항을 Decentralized Ledger System과 CAS에 저장하고 DID State에 대한 조회가 필요하다면 Decentralized Ledger System과 CAS로 부터 필요한 데이터를 불러와 사용자에게 전달 할 것입니다.

## DID URI Composition

![Sidetree based DID](https://i.imgur.com/uHm3fQI.png)

Sidetree 기반의 DID는 Short-Form, Long-form 2가지로 나뉘어 집니다.
Short-form DID는 `did:{METHOD}:{DID-SUFFIX}` 의 형식으로 이루어져 있으며 Long-form DID는 `did:{METHOD}:{DID-SUFFIX}:{LONG-FORM-SUFFIX-DATA}` 로 이루어져 있습니다.

![Long form DID Data](https://i.imgur.com/YbkJsj0.png)

`Long-Form Suffix Data`는 위 사진에 나온 `Long-form DID Data`를 Base64URL 형식으로 인코딩한 값입니다. 해당 값을 구성하는 필드를 어떻게 생성하는 지는 DID Operation의 Create에서 자세히 설명 할 것입니다.

## DID Operation

![DID Operation](https://i.imgur.com/KdfHxwQ.png)

Sidetree 프로토콜에서 수행할 수 있는 DID Operation은 크게 Create, Update, Recover, Deactivate의 4가지 입니다.

### Create

Create Operation은 DID를 생성하는 기능을 수행하며 이후 동작에 사용할 `Update Key Pair`와 `Recovery Key Pair`를 생성하는 과정을 포함하고 있습니다.

이 과정에서 `Long form DID Data`를 구성하는 데이터들을 생성하게 되고 여기서 생성한 값들을 `Long-form DID`를 구성하는데 사용합니다.

![Create](https://i.imgur.com/l4R9bPL.png)

### Update

Update Operation은 해당 DID에 대한 attribute를 수정하는 기능을 수행합니다. (eg. add/remove keys, add/remove services, ...)

Create Operation 혹은 이전 Update Operation 과정을 수행하면서 만든 Update key를 필요로 하며 동일한 키를 계속 사용하는것이 아닌 새로운 Update key Pair인 `Next Update Key Pair`를 생성하여 다음 Operation에 사용할 수 있도록 합니다.

![Update](https://i.imgur.com/ToxFJd9.png)

### Recover

Recover Operation은 해당 DID에 대한 attribute를 복구하는 기능을 수행합니다.

기존에 생성한 Update key, Recovery Key 모두를 필요로 하며 Update와 마찬가지로 이후 Operation에서는 동일한 키를 계속 사용하는것이 아닌 새로운 Update key Pair인 `Next Update Key Pair`, 새로운 Recovery key Pair인 `Next Recovery Key Pair`를 생성하여 다음 Operation에 사용할 수 있도록 합니다.

![Recover](https://i.imgur.com/VOVZPyH.png)

### Deactivate

Deactivate는 해당 DID를 비활성화 하는 기능을 수행합니다.

![Deactivate](https://i.imgur.com/Px9BfJ7.png)

## Patches

DID Operation를 수행하면서 Delta Object를 생성하는 과정에서 `Patches` 라는 필드가 Array로 포함되는데 이 필드에 실제 어떤 기능을 하는지에 대한 내용이 포함됩니다. 

Sidetree 프로토콜에서 제공하는 Patches에 대한 종류는 아래 사진과 같습니다.

![Patches](https://i.imgur.com/VxU1Zq3.png)

자세한 내용은 [DID State Patches](https://identity.foundation/sidetree/spec/#did-state-patches) 링크를 참조하세요

## Reference

[Sidetree DIF spec](https://identity.foundation/sidetree/spec/)

[ION github](https://github.com/decentralized-identity/ion)