---
layout: post
title: "ERC-2535 Diamond 패턴의 실제 구현과 보일러플레이트 소개"
date: "2025-03-17 11:30:00 +0900"
tags: [blockchain]
---

안녕하세요! [지난 포스팅](/2025/03/14/erc-2535-diamond/)에서 Diamond 패턴의 기본 개념과 구조에 대해 설명드렸는데요. 오늘은 제가 직접 구현한 Diamond 패턴 보일러플레이트를 통해 실제 구현 방법과 사용 예시를 자세히 살펴보도록 하겠습니다.

![Diamond 패턴 구조](/assets/post/2025-03-14-erc-2535-diamond/diamond_pattern.svg)
*ERC-2535 Diamond 패턴의 구조*

## 1. 보일러플레이트 소개

이번 포스팅에서 사용하는 보일러플레이트 전체 소스 코드는 [GitHub](https://github.com/sweatpotato13/solidity-contract-boilerplate)에서 확인하실 수 있습니다. 

이 보일러플레이트는 ERC-2535 Diamond 패턴을 구현하는데 필요한 모든 기본 구성요소를 포함하고 있습니다. 주요 특징은 다음과 같습니다:

- 기본 Diamond 컨트랙트 구현
- 다양한 Facet 예제 (ERC20, Counter, Calculator 등)
- Diamond Cut/Loupe 기능 구현
- 업그레이드 가능한 스마트 컨트랙트 구조
- 테스트 코드 포함


## 2. 프로젝트 구조

![Boilerplate 구조](/assets/post/2025-03-17-erc-2535-diamond-implementation/boilerplate_structure.svg)
*Diamond 패턴 보일러플레이트 프로젝트 구조*

## 3. 핵심 구현 설명

### 3.1 Diamond 컨트랙트

Diamond 컨트랙트는 모든 함수 호출의 진입점 역할을 합니다. 전체 구현은 다음과 같습니다:

```solidity
// SPDX-License-Identifier: MIT
pragma solidity ^0.8.0;

/******************************************************************************\
* Author: Nick Mudge <nick@perfectabstractions.com> (https://twitter.com/mudgen)
* EIP-2535 Diamonds: https://eips.ethereum.org/EIPS/eip-2535
*
* Implementation of a diamond.
/******************************************************************************/

import {LibDiamond} from "./libraries/LibDiamond.sol";
import {IDiamondCut} from "./interfaces/IDiamondCut.sol";

contract Diamond {
    constructor(address _contractOwner, address _diamondCutFacet) payable {
        LibDiamond.setContractOwner(_contractOwner);

        // Add the diamondCut external function from the diamondCutFacet
        IDiamondCut.FacetCut[] memory cut = new IDiamondCut.FacetCut[](1);
        bytes4[] memory functionSelectors = new bytes4[](1);
        functionSelectors[0] = IDiamondCut.diamondCut.selector;
        cut[0] = IDiamondCut.FacetCut({
            facetAddress: _diamondCutFacet,
            action: IDiamondCut.FacetCutAction.Add,
            functionSelectors: functionSelectors
        });
        LibDiamond.diamondCut(cut, address(0), "");
    }

    // Find facet for function that is called and execute the
    // function if a facet is found and return any value.
    fallback() external payable {
        LibDiamond.DiamondStorage storage ds;
        bytes32 position = LibDiamond.DIAMOND_STORAGE_POSITION;
        // get diamond storage
        assembly {
            ds.slot := position
        }
        // get facet from function selector
        address facet = ds.selectorToFacetAndPosition[msg.sig].facetAddress;
        require(facet != address(0), "Diamond: Function does not exist");
        // Execute external function from facet using delegatecall and return any value.
        assembly {
            // copy function selector and any arguments
            calldatacopy(0, 0, calldatasize())
            // execute function call using the facet
            let result := delegatecall(gas(), facet, 0, calldatasize(), 0, 0)
            // get any return value
            returndatacopy(0, 0, returndatasize())
            // return any return value or error back to the caller
            switch result
            case 0 {
                revert(0, returndatasize())
            }
            default {
                return(0, returndatasize())
            }
        }
    }

    receive() external payable {}
}
```

### 3.2 Facet 업그레이드 실습: Counter Facet

이 보일러플레이트에서는 Counter Facet의 세 가지 버전을 통해 다이아몬드 패턴에서의 업그레이드 과정을 실습할 수 있습니다.

![Facet 버전 업그레이드](/assets/post/2025-03-17-erc-2535-diamond-implementation/facet_upgrade_process.svg)
*CounterFacet 버전별 업그레이드 과정*

#### 3.2.1 CounterFacet (V1)

기본 Counter Facet 구현은 다음과 같습니다:

```solidity
// SPDX-License-Identifier: MIT
pragma solidity ^0.8.0;

import "../libraries/LibDiamond.sol";
import "../libraries/LibCounter.sol";

contract CounterFacet {
    // Access to counter storage
    function getCounterStorage() internal pure returns (LibCounter.CounterStorage storage) {
        return LibCounter.counterStorage();
    }

    // Get the current counter value
    function getCount() external view returns (uint256) {
        return getCounterStorage().count;
    }

    // Increment the counter
    function increment() external {
        getCounterStorage().count += 1;
    }

    // Decrement the counter
    function decrement() external {
        LibCounter.CounterStorage storage cs = getCounterStorage();
        require(cs.count > 0, "Count cannot be negative");
        cs.count -= 1;
    }

    // Set the counter to a specific value
    function setCount(uint256 _count) external {
        LibDiamond.enforceIsContractOwner(); // Only owner can set count
        getCounterStorage().count = _count;
    }
}
```

스토리지는 LibCounter.sol에 정의되어 있습니다:

```solidity
// SPDX-License-Identifier: MIT
pragma solidity ^0.8.0;

/**
 * @title LibCounter
 * @dev Storage library for Counter facet
 */
library LibCounter {
    struct CounterStorage {
        uint256 count;
    }

    // Position in storage is determined by keccak256 of a unique string
    bytes32 constant COUNTER_STORAGE_POSITION = keccak256("diamond.counter.storage");

    // Returns the struct from a specified position in contract storage
    function counterStorage() internal pure returns (CounterStorage storage cs) {
        bytes32 position = COUNTER_STORAGE_POSITION;
        assembly {
            cs.slot := position
        }
    }
}
```

#### 3.2.2 CounterFacetV2 (기능 추가)

V2에서는 새로운 기능과 이벤트를 추가하였습니다:

```solidity
// SPDX-License-Identifier: MIT
pragma solidity ^0.8.0;

import "../libraries/LibDiamond.sol";
import "../libraries/LibCounter.sol";

/**
 * @title CounterFacetV2
 * @dev Upgraded version of CounterFacet with additional functionality
 */
contract CounterFacetV2 {
    // Access to counter storage
    function getCounterStorage() internal pure returns (LibCounter.CounterStorage storage) {
        return LibCounter.counterStorage();
    }

    // Get the current counter value
    function getCount() external view returns (uint256) {
        return getCounterStorage().count;
    }

    // Increment the counter
    function increment() external {
        getCounterStorage().count += 2;

        // Added in V2: Emit event when counter is incremented
        emit CounterIncremented(getCounterStorage().count);
    }

    // Decrement the counter
    function decrement() external {
        LibCounter.CounterStorage storage cs = getCounterStorage();
        require(cs.count > 0, "Count cannot be negative");
        cs.count -= 2;

        // Added in V2: Emit event when counter is decremented
        emit CounterDecremented(cs.count);
    }

    // Set the counter to a specific value
    function setCount(uint256 _count) external {
        LibDiamond.enforceIsContractOwner(); // Only owner can set count
        uint256 oldCount = getCounterStorage().count;
        getCounterStorage().count = _count;

        // Added in V2: Emit event when counter value is changed
        emit CounterSet(oldCount, _count);
    }

    // Added in V2: New function to increment counter by 2
    function doubleIncrement() external {
        getCounterStorage().count += 2;
        emit CounterIncremented(getCounterStorage().count);
    }

    // Added in V2: Function to check if counter is a multiple of a given number
    function isMultipleOf(uint256 _number) external view returns (bool) {
        require(_number != 0, "Cannot divide by zero");
        return getCounterStorage().count % _number == 0;
    }

    // Events added in V2
    event CounterIncremented(uint256 newCount);
    event CounterDecremented(uint256 newCount);
    event CounterSet(uint256 oldCount, uint256 newCount);
}
```

V2에서는 다음 내용이 추가되었습니다:
- increment/decrement 함수가 2씩 증가/감소하도록 수정
- 이벤트 추가 (CounterIncremented, CounterDecremented, CounterSet)
- 새로운 함수 추가 (doubleIncrement, isMultipleOf)

#### 3.2.3 CounterFacetV3 (스토리지 확장)

V3에서는 스토리지 구조를 확장했습니다. 이를 위해 새로운 라이브러리(LibCounterV2.sol)를 사용합니다:

![스토리지 확장 과정](/assets/post/2025-03-17-erc-2535-diamond-implementation/storage_extension.svg)
*V3 업그레이드에서 스토리지 확장 과정*

```solidity
// SPDX-License-Identifier: MIT
pragma solidity ^0.8.0;

/**
 * @title LibCounterV2
 * @dev Storage library for CounterFacet with extended functionality
 */
library LibCounterV2 {
    struct CounterStorage {
        uint256 count;
        uint256 lastIncremented; // Last increment timestamp
        uint256 lastDecremented; // Last decrement timestamp
        uint256 totalIncrements; // Total number of increments
        uint256 totalDecrements; // Total number of decrements
        address lastModifier; // Address of the last modifier
    }

    // Position in storage is determined by keccak256 of a unique string - Keep the same key as original
    bytes32 constant COUNTER_STORAGE_POSITION = keccak256("diamond.counter.storage");

    // Returns the struct from a specified position in contract storage
    function counterStorage() internal pure returns (CounterStorage storage cs) {
        bytes32 position = COUNTER_STORAGE_POSITION;
        assembly {
            cs.slot := position
        }
    }

    // Storage migration function
    function migrateStorage() internal {
        CounterStorage storage cs = counterStorage();

        // Initialize data if not already initialized
        if (cs.lastModifier == address(0)) {
            cs.lastIncremented = block.timestamp;
            cs.lastDecremented = block.timestamp;
            cs.totalIncrements = 0;
            cs.totalDecrements = 0;
            cs.lastModifier = msg.sender;
        }
    }
}
```

CounterFacetV3 구현:

```solidity
// SPDX-License-Identifier: MIT
pragma solidity ^0.8.0;

import "../libraries/LibDiamond.sol";
import "../libraries/LibCounterV2.sol";

/**
 * @title CounterFacetV3
 * @dev Version of CounterFacet that uses extended storage structure
 */
contract CounterFacetV3 {
    // Access to counter storage
    function getCounterStorage() internal pure returns (LibCounterV2.CounterStorage storage) {
        return LibCounterV2.counterStorage();
    }

    // Function for initial migration
    function initializeV3() external {
        LibDiamond.enforceIsContractOwner(); // Only owner can initialize
        LibCounterV2.migrateStorage();
    }

    // Get the current counter value
    function getCount() external view returns (uint256) {
        return getCounterStorage().count;
    }

    // Increment the counter with extended functionality
    function increment() external {
        LibCounterV2.CounterStorage storage cs = getCounterStorage();
        cs.count += 3;
        cs.lastIncremented = block.timestamp;
        cs.totalIncrements += 1; // Single operation
        cs.lastModifier = msg.sender;

        emit CounterIncremented(cs.count, cs.totalIncrements, msg.sender);
    }

    // ... 그 외 다양한 함수들 ...

    // Added in V3: Function to get extended counter information
    function getCounterInfo()
        external
        view
        returns (
            uint256 count,
            uint256 lastIncremented,
            uint256 lastDecremented,
            uint256 totalIncrements,
            uint256 totalDecrements,
            address lastModifier
        )
    {
        LibCounterV2.CounterStorage storage cs = getCounterStorage();
        return (
            cs.count,
            cs.lastIncremented,
            cs.lastDecremented,
            cs.totalIncrements,
            cs.totalDecrements,
            cs.lastModifier
        );
    }

    // Events added in V3
    event CounterIncremented(uint256 newCount, uint256 totalIncrements, address modifierAddress);
    event CounterDecremented(uint256 newCount, uint256 totalDecrements, address modifierAddress);
    event CounterSet(uint256 oldCount, uint256 newCount, address modifierAddress);
}
```

V3의 주요 변경점:
- 확장된 스토리지 구조 사용 (카운터 값 + 추가 메타데이터)
- 스토리지 마이그레이션 함수 (initializeV3)
- 새로운 함수들 추가 (getCounterInfo, getLastIncrementTime, 등)
- 이벤트에 더 많은 정보 포함

### 3.3 Facet 업그레이드 프로세스

업그레이드 과정은 스크립트를 통해 자동화되어 있습니다. 주요 과정은 다음과 같습니다:

![다이아몬드 컷 과정](/assets/post/2025-03-17-erc-2535-diamond-implementation/diamond_cut_process.svg)
*Diamond Cut 과정을 통한 Facet 업그레이드 메커니즘*

#### V1 → V2 업그레이드 (기능 업그레이드)

```typescript
// upgrade-counter-v2-facet.ts
async function main() {
    // 새 Facet 배포
    const CounterFacetV2 = await ethers.getContractFactory("CounterFacetV2");
    const counterFacetV2 = await CounterFacetV2.deploy();
    
    // Diamond Cut Facet 인스턴스화
    const diamondCutFacet = await ethers.getContractAt("IDiamondCut", diamondAddress);
    
    // 기존 함수 셀렉터 가져오기
    const selectors = getSelectors(counterFacet);
    
    // 새 함수 셀렉터 가져오기
    const selectorsV2 = getSelectors(counterFacetV2);
    
    // 1단계: 기존 CounterFacet 기능 제거
    const cutRemove = {
        facetAddress: ethers.ZeroAddress,
        action: FacetCutAction.Remove,
        functionSelectors: selectors,
    };
    
    // 2단계: 새 CounterFacetV2 기능 추가
    const cutAdd = {
        facetAddress: counterFacetV2Address,
        action: FacetCutAction.Add,
        functionSelectors: selectorsV2,
    };
    
    // 업그레이드 실행 (제거 후 추가)
    await diamondCutFacet.diamondCut([cutRemove], ethers.ZeroAddress, "0x");
    await diamondCutFacet.diamondCut([cutAdd], ethers.ZeroAddress, "0x");
}
```

#### V2 → V3 업그레이드 (스토리지 업그레이드)

```typescript
// upgrade-counter-v3-facet.ts
async function main() {
    // 현재 카운터 값 확인
    const currentCount = await counterFacetV2.getCount();
    
    // 새 Facet 배포
    const CounterFacetV3 = await ethers.getContractFactory("CounterFacetV3");
    const counterFacetV3 = await CounterFacetV3.deploy();
    
    // 기존 V2 함수 셀렉터 가져오기
    const selectorsV2 = getSelectors(CounterFacetV2);
    
    // 새 V3 함수 셀렉터 가져오기
    const selectorsV3 = getSelectors(counterFacetV3);
    
    // 1단계: 기존 CounterFacetV2 기능 제거
    const cutRemove = {
        facetAddress: ethers.ZeroAddress,
        action: FacetCutAction.Remove,
        functionSelectors: selectorsV2,
    };
    
    // 2단계: 새 CounterFacetV3 기능 추가
    const cutAdd = {
        facetAddress: counterFacetV3Address,
        action: FacetCutAction.Add,
        functionSelectors: selectorsV3,
    };
    
    // 업그레이드 실행 (제거 후 추가)
    await diamondCutFacet.diamondCut([cutRemove], ethers.ZeroAddress, "0x");
    await diamondCutFacet.diamondCut([cutAdd], ethers.ZeroAddress, "0x");
    
    // 스토리지 마이그레이션 실행
    const counterFacetV3Proxy = await ethers.getContractAt("CounterFacetV3", diamondAddress);
    await counterFacetV3Proxy.initializeV3();
    
    // 업그레이드 확인
    const count = await counterFacetV3Proxy.getCount();
    console.log("업그레이드 후 카운터 값 (보존되어야 함):", count.toString());
}
```

## 4. 보일러플레이트 사용 방법

1. 저장소 클론
```bash
git clone https://github.com/sweatpotato13/solidity-contract-boilerplate.git
```

2. 의존성 설치
```bash
pnpm install
```

3. 테스트 실행
```bash
pnpm test
```

4. 로컬 네트워크에 배포
```bash
pnpm node
pnpm deploy:local
```

5. Facet 업그레이드 테스트

```bash
# V1 → V2 업그레이드
npx hardhat run scripts/upgrade-counter-v2-facet.ts --network localhost 

# V2 → V3 업그레이드 (스토리지 확장)
npx hardhat run scripts/upgrade-counter-v3-facet.ts --network localhost
```

![전체 워크플로우](/assets/post/2025-03-17-erc-2535-diamond-implementation/diamond_workflow.svg)
*Diamond 패턴 개발 및 업그레이드 워크플로우*

## 마무리

이번 포스팅에서는 Diamond 패턴의 실제 구현과 보일러플레이트 사용 방법에 대해 알아보았습니다. 특히 Facet 업그레이드와 스토리지 확장 과정을 통해 다이아몬드 패턴의 강력한 유연성을 확인할 수 있었습니다.

이 보일러플레이트를 활용하면 기능 추가, 버그 수정, 스토리지 확장 등 다양한 업그레이드 시나리오를 쉽게 구현할 수 있습니다. 이는 복잡한 스마트 컨트랙트 시스템을 개발하고 유지보수하는 데 큰 도움이 될 것입니다.