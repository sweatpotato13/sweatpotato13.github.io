---
layout: post
title: "Optimism fault proof 알아보기"
date: "2024-08-29 10:56:44 +0900"
tags: [blockchain]
---

# Fault Proofs

https://docs.optimism.io/stack/protocol/fault-proofs/explainer

https://specs.optimism.io/fault-proof/cannon-fault-proof-vm.html

---

# Overview

이 문서는 Fault Proof 관련 내용을 스터디 하여 관련 내용을 작성한 문서입니다. 이 문서는 Optimism 에서 작성된 문서를 참고하여 작성하였습니다.

# Fault Proof Explainer

Fault Proofs는 Optimistic Rollup 시스템에서 중요한 부분입니다. Fault Proof는 상태에 대한 제안을 허가없이(Permissionless) 제출하고 이를 challenge할 수 있습니다.

## Permissionless Proposals

`Proposals` 혹은 `State Proposals` 는 `DisputeGameFactory` contract를 통해 제출된 체인의 상태에 대한 클레임 입니다. `Proposals`는 여러 용도로 사용할 수 있지만 일반적으로 최종 사용자가 체인에서 한 행동에 대하여 증명하는데 일반적으로 사용됩니다.

Fault Proofs는 이러한 `Proposals`를 누구나 제출할 수 있으며 아무나에 의해 확정될 수 있습니다.

![image.png](/assets/post/fault-proof/image.png)

## Permissionless Challenges

누구나 `Proposals` 를 제출할 수 있기 때문에 잘못된 Proposals에 이의를 제기할 수 있는 것이 중요합니다. Optimistic Rollup에서는 사용자의 Proposals가 잘못되었다고 생각하는 경우 이의를 제기할 수 있는 기간이 있습니다. 이 이의는 권한이 필요하지 않으며 누구나 제출할 수 있습니다. 모든 사용자는 체인에 대한 노드를 실행하고 challenger 도구를 사용하여 이러한 분쟁 프로세스에 참여할 수 있습니다.

# FP System Components

Fault Proof System은 `Fault Proof Program (FPP)`, `Fault Proof Virtual Machine (FPVM)`, `dispute game protocol` 의 3가지로 구성됩니다. 이러한 구성 요소는 네트워크에서 악의적이거나 오류가 있는 활동에 challenge를 하여 시스템 내의 신뢰와 일관성을 유지합니다.

## **Fault Proof Program**

FPP의 주요 기능은 L1 입력에서 L2 출력을 검증하기 위해 `rollup state-transition` 을 실행합니다. 이 검증 가능한 출력은 L1 의 분쟁이 있는 출력을 해결하는데 사용될 수 있습니다.

FPP는 `op-node` (Consensus Layer) 와 `op-geth` (Execution Layer) 의 결합으로 구성되어 있으며 이 같은 구성 때문에 단일 프로세스에서 consensus와 execution의 일부를 모두 가지고 있습니다. 이 는 일반적으로 `op-geth` 를 호출할 때 HTTP를 통해 호출되는 것이 아니라 직접적인 메소드 호출이 가능하다는 것입니다.

FPP는 동일한 입력 데이터를 사용하여 두번 호출 해도 동일한 출력 뿐만 아니라 동일한 프로그램 실행 추적이 가능하도록 결정론적 방식으로 실행할 수 있도록 설계 되었습니다. 이를 통해 분쟁 해결 프로세스의 일부를 온체인 VM 에서 실행이 가능합니다.

모든 데이터는 [Preimage Oracle](https://specs.optimism.io/fault-proof/index.html#pre-image-oracle) API를 통해서 가져오며 Preimage 는 온체인일때 FPVM을 통해 제공되거나 혹은 node에서 필요한 데이터를 다운로드할 수 있는 기본 host 구현을 통해 제공될 수 있습니다.

## **Fault Proof Virtual Machine**

FPVM은 FPP와 분리되어 구현되어 있습니다. 이 같은 특성 때문에 두 컴포넌트는 병렬적으로 업그레이드가 가능합니다. 

FPVM 내에서 실행되는 FPP (Client-side)는 L2 state-transition 을 표현하는 부분이며 FPVM과 FPP간 인터페이스는 표준화되어 [문서](https://github.com/ethereum-optimism/optimism/blob/546fb2c7a5796b7fe50b0b7edc7666d3bd281d6f/specs/cannon-fault-proof-vm.md)화 되어 있습니다.

분리되어 구현하였기 때문에 FPVM은 최소한의 구현만으로 유지됩니다. EVM op-code 추가와 같은 Ethereum 프로토콜에 대한 변경은 FPVM에 영향을 끼치지않습니다. 대신 프로토콜이 변경되면 FPP를 업데이트하여 새로운 state-transition을 적용해 주어야 합니다.

FPVM은 lower-level 의 명령어 실행을 담당합니다. 또한 FPP를 에뮬레이션 해야합니다.  

FPVM에 대한 요구사항은 낮습니다. 프로그램은 동기식이며 모든 입력은 동일한 Preimage Oracle 을 통해 로딩 되지만 이 모든 것은 L1 온체인에서 증명 되어야 합니다. 이를 위해 한번에 하나의 명령어만 증명됩니다. 

`Bisection game` 은 full execution trace 를 증명하는 것에서 single instruction 만을 증명하는 것으로 작업을 축소시킬 수 있습니다. 

instruction을 증명하는 것은 FPVM 마다 다를 수 있지만, 대부분은 [Cannon](https://docs.optimism.io/stack/protocol/fault-proofs/cannon) 과 비슷합니다.

- 명령을 실행하기 위해 VM은 therad-context와 유사한것은 에뮬레이션 합니다. 명령어는 메모리에서 읽고 해석되며 레지스터 파일과 메모리는 약간 변경될 수 있습니다.
- Preimage Oracle과 기본 프로그램 런타임 요구사항을 지원하기 위해 Linux 시스템 호출의 subset도 지원합니다. Read/Write Syscall은 Preimage Oracle과의 상호작용을 가능하게 합니다. 프로그램은 Preimage에 대한 요청으로 hash를 쓰고 한번에 작은 청크들로 값을 읽습니다.

[Cannon](https://docs.optimism.io/stack/protocol/fault-proofs/cannon)은 Optimism의 모든 분쟁에서 사용하는 기본 FPVM이며 [MIPS](https://docs.optimism.io/stack/protocol/fault-proofs/mips)는 dispute game의 모듈성으로 인해 구현될 수 있는 Cannon의 온체인 Smart contract 구현입니다.

## **Dispute Game Protocol**

Dispute protocol 에서 다양한 타입의 dispute games은 [DisputeGameFactory](https://github.com/ethereum-optimism/optimism/blob/6a53c7a3294edf140d552962f81c0f742bf445f9/packages/contracts-bedrock/src/dispute/DisputeGameFactory.sol#L4)를 통해 생성, 관리 및 업그레이드 할 수 있습니다.

dispute games는 dispute protocol의 핵심 기본 요소입니다. 이는 간단한 state machine을 모델링하며, 분쟁을 제기할 수 있는 모든 정보에 대한 32byte commitment를 initialize 할 수 있습니다. 이 commitment에 대해서 true/false로 resolve하는 기능이 포함되어 있으며 이는 구현자가 정의하도록 남겨둡니다. dispute games은 두가지 기본 속성에 의존합니다.

- Incentive Compatibility: 이 시스템은 false claim에 대해 페널티를 제공하고 truthful claim에 대해서는 합당한 보상을 제공합니다.
- Resolution: 각 game은 claim에 대해서 확실하게 validate 한지 invalidate한지 결정할 수 있는 메커니즘이 존재합니다.

Optimism에서의 표준은 `bisection game` 이며 이는 dispute game의 한 종류입니다.  

# **Fault Proof VM: Cannon**

Cannon은 Optimism에서 표준으로 사용하는 Fault Proof Virtual Machine(FPVM)의 인스턴스입니다. Cannon에는 두 가지 주요 구성 요소가 있습니다.

- Onchain `MIPS.sol`: 단일 MIPS의 명령어 실행을 검증하는 EVM 구현체
- Offchain `mipsevm`: MIPS 명령어가 온체인에서 검증할 수 있도록 Proof를 생성하는 Go 구현체

## Control Flow

![image.png](/assets/post/fault-proof/image1.png)

### **OP-Challenger ↔ Cannon**

active fault dispute game이 L2 block state transitions보다 낮은 depth에 도달하면 `op-challenger` 는 `Cannon` 을 실행하여 FPVM 내에서 MIPS 명령어 처리를 시작합니다. MIPS 명령어를 처리하는 과정에서 `Cannon` 은 FPVM 내에서 MIPS 명령어의 계산 결과에 대한 commitment인 state witness hash를 생성합니다. Bisection game 에서 `op-challenger` 는 active dispute를 식별할 수 있는 단일 MIPS 명령어가 될 때 까지 hash 를 생성해 제공합니다. 그러면 `Cannon` 은 MIPS 명령어를 onchain에서 실행하는데 필요한 모든 정보가 포함된 witness proof를 생성합니다. MIPS.sol에서 이 단일 MIPS 명령어를 onchain에서 실행하면 fault dispute game를 증명할 수 있습니다.

### **OP-Program ↔ Cannon**

execution trace bisection이 시작되고 `Cannon` 이 시랭되면 Executable and Linkable Format (ELF) 바이너리가 로드되어 `Cannon` 내에서 실행됩니다. `Cannon` 내에는 MIPS R3000, 32비트 명령어 집합 아키텍처(ISA)를 처리하도록 빌드된 `mipsevm`이 있습니다. ELF 파일에는 MIPS 명령어가 포함되어 있으며 MIPS 명령어로 컴파일된 코드는 `op-program` 입니다.

`op-program` 은 MIPS 명령어로 컴파일되어 `Cannon` FPVM 내에서 실행되는 golang 코드 입니다. `op-program` 은 golang 코드 자체로 실행하든 `Cannon` 에서 실행하든 L2 state를 도출하는데 사용되는 모든 필수 데이터를 가져옵니다. 

`op-program`은 동일한 입력이 동일한 출력 뿐만 아니고 동일한 execution traces을 생성하도록 설계 되었습니다. 이를 통해 fault dispute game의 모든 참가자가 `op-program` 을 실행하여 동일한 L2 oitput root state transition이 주어지면 동일한 execution traces를 생성할 수 있습니다. 이를 통해 체인에서 실행될 동일한 MIPS 명령어에 대해 동일한 witness proof가 생성될 수 있습니다.

## **Overview of Offchain Cannon components**

![image.png](/assets/post/fault-proof/image2.png)

### **`mipsevm` State and Memory**

`mipsevm` 은 32-bit이기 때문에 주소 범위는 `[0, 2^32-1]`을 갖습니다. 메모리 구조는 일반적인 monolithic 구조이며 VM은 물리적 메모리와 직접 상호작용하는것 처럼 동작합니다.

`mipsevm` 의 경우 메모리가 어떻게 저장 되는지 중요하지 않습니다. Go runtime 내에서 전체 메모리를 보관할 수 있기 때문입니다. 하지만 MIPS 명령어를 온체인에서 실행하기 위해서는 작은 부분만 필요하도록 메모리를 표현하는 것이 중요합니다. 이는 비용 때문에 온체인에서 전체 32bit 메모리 공간을 전부 표현하는 것이 불가능 하기 때문입니다. 따라서 메모리는 Binary Merkel tree 구조에 저장됩니다. tree는 27 level의 고정된 depth를 가지며 각 Leaf 는 32byte입니다. 이는 전체 32bit 주소 공간을 포함할 수 있습니다.

**Generating the Witness Proof**

`Cannon` 은 dispute game에서 두가지 주요 구성 요소를 처리합니다. 

- execution trace bisection game 중 `op-challenger` 에게 제공할 state witness hashes 생성
- single MIPS instruction 에 대한 witness proof 생성
- [witness.go](https://github.com/ethereum-optimism/optimism/blob/develop/cannon/cmd/witness.go) 에서 MIPS 명령어 관련 정보를 인코딩하여 MIPS.sol에 전달할 수 있는 형태로 준비합니다.
- Pre-image가 필요한 경우, 관련 키와 오프셋 정보를 OP-Challenger에 전하여 PreimageOracle.sol에 onchain으로 게시할 수 있도록 합니다.

### **Loading the ELF file**

bisection game의 execution trace 부분이 시작되면 MIPS 명령어로 컴파일된 `op-program` 이 포함된 ELF 파일이 `Cannon` 내에서 실행됩니다. 그러나 `op-program`을 `Cannon` 으로 가져와 실행하려면 바이너리 로더가 필요합니다. 바이너리 로더는 `load_elf.go`와 `patch.go`로 구성됩니다.

`load_elf.go` 는 최상위 arguments를 분석하고 ELF 바이너리를 읽어 `Cannon` 에서 실행할 수 있게 합니다.

`patch.go` 는 ELF 파일의 헤더를 실제로 파싱 하는 역할을 하는데 이 헤더는 파일 내 어떤 프로그램이 있는지 메모리의 어느 위치에 있는지를 지정합니다.

ELF 파일을 로드하는 동안 발생하는 또 다른 단계는 호환되지 않는 함수의 바이너리를 패치하는 것 입니다. EVM 에서는 커널에 접근이 불가능하므로 Onchain, Offchain의 구현을 일치시켜야 하기 위해서 Offchain에서의 커널 엑세스도 `Cannon`내에서는 사용할 수 없도록 해야합니다.

### **Instruction Stepping**

MIPS 바이너리가 `Cannon` 에 로드 되면 MIPS 명령어를 한 번에 하나씩 실행할 수 있습니다. run.go에는 MIPS 명령어를 단계별로 실행하는 코드가 포함되어 있습니다  또한 각 명령어 실행 전 추가 작업(로깅, 스냅샷 등)을 수행할 수 있습니다. `run.go` 내에서 `StepFn`은 실행할 MIPS 명령어를 시작하는 Wrapper 입니다. [`instrumented.go`](https://github.com/ethereum-optimism/optimism/blob/develop/cannon/mipsevm/singlethreaded/instrumented.go)는 각 MIPS 명령어에 대해 시작할 인터페이스로 Step을 구현합니다. 또한 `instrumented.go`는 증인 증명에 대한 인코딩 정보와 사전 이미지 정보(MIPS 명령어에 필요한 경우)를 처리합니다.

### **`mips.go`**

**`mips.go` vs. `MIPS.sol`**

Offchain `mips.go`와 Onchains Cannon `MIPS.sol`은 32비트 MIPS III 명령어를 실행할 때 비슷하게 동작합니다. 동일한 명령어, 메모리 및 레지스터 상태가 주어지면 정확히 동일한 결과를 생성해야 합니다. 하지만 두 구현체 사이엔 차이점이 존재합니다.

1. `MIPS.sol`은 Onchain에서 실행되고 mips.go는 오프체인에서 실행됩니다.
    - `MIPS.sol`은 한번에 단일 명령어를 실행
    - `mips.go`는 한번에 모든 MIPS 명령어를 실행
2. `mipsevm` 은 전체 32bit monolithic 메모리 공간을 포함하고 MIPS 명령어의 결과에 따라 메모리의 상태를 유지 관리 하지만 `MIPS.sol` 은 상태가 없고 전체 메모리 공간을 유지 관리하지 않습니다.
3. mips.go는 PreimageOracle Server에 Preimage를 업로드를 해야합니다. 하지만 `MIPS.sol`은 그렇지 않습니다.

### **Preimage Oracle Interaction**

`Cannon`은 PreimageOracle 서버와 상호작용하여 필요한 Pre-image를 관리합니다. MIPS 시스템 호출 명령어 실행 시 Pre-image 읽기/쓰기를 수행합니다. 필요한 경우 `OP-Challenger`에게 온체인 `PreimageOracle.sol`에 데이터를 제공하도록 지시합니다.

# **OP-Challenger Explainer**

`op-challenger` 는 fault dispute system에서 *honest actor*로 동작하고 체인을 보호하고 dispute game이 항상 체인의 올바른 상태로 해결되도록 합니다. 구체적으로 `op-challenger` 는 아래와 같은 작업을 수행합니다.

- dispute game을 모니터링 하고 상호작용함
- proposal에 대한 valid output를 지킴
- proposal에 대한 invalid output에 대해 challenge를 시도
- claim을 resolve 함으로 써 `op-proposer` 을 지원
- challenger and proposer 에 대한 자금 청구

## Architecture

아래 그림은 `op-challenger` 가 수행하는 행동에 대한 다이어그램입니다.

![image.png](/assets/post/fault-proof/image3.png)

## **Fault Detection Responses**

`op-challenger` 는 각 claim의 validity를 체크하여 유효하지 않다고 생각되는 claim에만 응답을 하는데 이는 아래 로직을 따릅니다.

1. trusted node가 동의하면 `op-challenger` 는 아무런 조치도 취하지 않습니다. 정직한 challenger는 disagree가 없는 클레임에 대해서 아무런 행동을 하지 않습니다.
2. trusted node가 동의하지 않는다면 `op-challenger` 는 제한된 output에 대해 counter-claim을 제시합니다. 정직한 challenger는 rollup node가 canonical state와 동기화 되고 fault proof program이 올바르다고 가정하므로 모든 오류에 대응하기 위해 돈을 걸 수 있습니다.

## **Fault Dispute Game Responses**

`op-challenger` 는 contract에 저장된 claim을 반복하여 조상이 자식보다 먼저 처리되도록 합니다. 각 클레임에 대해 정직한 challenger는 모든 claim에 대한 정직한 응답을 모두 추적하고 결정합니다.

### **Root Claim**

root claim은 root claim에 대한 정직한 challenger의 state witness hash가 있는 경우에만 정직한 claim으로 여겨집니다.

### **Counter Claims**

dispute game 에 새로운 claim이 생기면 정직한 challenger는 이를 처리하고 응답해야합니다. 정직한 challenger는 아래의 경우에만 응답을 합니다.

1. claim이 정직한 응답의 child인 경우
2. claim의 tract index보다 같거나 큰 trace index인 claim을 포함하는 정직한 응답

### **Possible Moves**

challenger는 새로운 claim이 추가될 때 마다 각 게임을 모니터링 [honest actor algorithm](https://specs.optimism.io/fault-proof/stage-one/honest-challenger-fdg.html)에 따라 유효한 제안은 통과시키고 유효하지 않은 제안은 차단합니다.

![image.png](/assets/post/fault-proof/image4.png)

## **Resolution**

한쪽의 `FaultDisputeGame` 의 Chess clock이 소진되면 정직한 challenger는 `FaultDisputeGame` 에 있는 `resolveClaim` 함수를 호출하는 것 입니다. root claim의 하위 game이 resolve되면 challenger는 마지막으로 `resolve` 함수를 호출하여 모든 game을 종료합니다.

`FaultDisputeGame` 는 resolve에 시간 제한을 두지 않습니다. 정직한 challenger는 그들이 반박한 claim에 붙은 자금 때문에 신속하게 해결하여 자금을 청산하고 보상을 확보하려는 경제적 동기를 갖습니다.

# **Fault Proof VM: MIPS.sol**

https://github.com/ethereum-optimism/optimism/blob/develop/packages/contracts-bedrock/src/cannon/MIPS.sol

MIPS.sol smart contract는 32비트, Big-Endian, MIPS III 명령어 집합 아키텍처(ISA)를 포함하는 가상 머신(VM)의 Onchain Implementation 입니다. 이 smart contract는 Offchain `mipsevm` 의golang Implementation과 동일합니다. 

## **Control Flow**

`FaultDisputeGame.sol`은 `MIPS.sol`과 상호 작용한 다음 `MIPS.sol`이 `PreimageOracle.sol`을 호출합니다. `MIPS.sol`은 누군가가 step을 호출해야 할 때 게임의 최대 depth에서만 호출됩니다. `FaultDisputeGame.sol`은 active dispute에 대한 Fault Dispute Game의 배포된 인스턴스이고 `PreimageOracle.sol`은 Pre-images를 저장합니다.

- Preimage는 L1과 L2 모두의 데이터를 포함하며, 여기에는 블록 헤더, 트랜잭션, 영수증, world state node 등의 정보가 포함됩니다. 이 Preimage는 실제 L2 상태를 계산하는 데 사용되는 유도 과정의 입력으로 활용되며, 이렇게 계산된 실제 L2 state는 이후 dispute game을 resolve 하는데 사용됩니다.
- Fault Dispute Game은 큰 틀에서 현재 합의된 L2 state를 결정하고, 첫 번째로 이견이 발생하는 state가 발견될 때까지 L2 state를 순차적으로 진행합니다.

`MIPS.sol` 컨트랙트는 진행 중인 dispute game 인스턴스, 즉 `FaultDisputeGame.sol` 컨트랙트에 의해 호출되며 dispute game이 현재 dispute가 진행중인 state transition tree의 leaf node에 도달한 후에만 호출 됩니다. Leaf node는 단일 MIPS 명령어를 나타내며 (FPVM으로 Cannon을 사용하는 경우) 이는 체인에서 실행될 수 있습니다.  이 명령어 이전까지 합의 된 L2 state인 Preimage와 MIPS.sol contract에서 실행할 명령어 state가 주어지면 fault dispute games는 알맞은 사후 state(혹은 Image)를 결정할 수 있습니다. 이는 이후에 disputer가 제안한 state와 비교하여 fault dispute games의 결과를 결정하는데 사용합니다.