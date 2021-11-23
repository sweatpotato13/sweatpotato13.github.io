---
layout: post
title: "Truffle을 사용하여 ERC-721 토큰 배포하기"
date: "2021-11-23 13:03:44 +0900"
tags: [tutorial, blockchain]
---

![Untitled](https://res.cloudinary.com/practicaldev/image/fetch/s--_fHAS3Uc--/c_imagga_scale,f_auto,fl_progressive,h_420,q_auto,w_1000/https://dev-to-uploads.s3.amazonaws.com/uploads/articles/jd23pnq93vw1fz9rlfks.png)

## About

최근 NFT에 대한 관심이 매우 뜨거워지고 있습니다. 우리가 알고있는 대부분의 NFT는 이더리움 네트워크 상에서 동작하고 있으며 이는 대부분 ERC-721 혹은 ERC-1155 형식의 토큰을 사용하고 있습니다. NFT에 주로 사용되는 ERC-721은 토큰의 수량을 주로 다루는 ERC-20과 다르게 토큰의 소유권(Token ID, Token owner)를 주로 다루고 있기 때문에 각각의 토큰에 대한 소유권을 소유자에게 부여하는 특징을 갖게 됩니다.

본 문서에서는 컴파일 및 배포를 가능하게 해주는 툴인 `Truffle`와 `openzeppelin` 패키지를 이용해서 이더리움 네트워크에서 사용할 수 있는 ERC-721 토큰을 배포하는 예제를 작성하였습니다.

본 문서에서는 메인넷 및 테스트넷의 이더리움 주소 및 ETH가 있다고 가정하고 진행하였습니다.

## Prerequisite

### Truffle 설치

아래 명령어로 `Truffle`을 설치해 줍니다. 

```bash
npm install -g truffle
```

### 프로젝트 생성

`Truffle` 로 사용 될 폴더를 만들고 그 안에서 `truffle init` 을 실행하여 초기 구성 파일을 생성합니다.

```bash
mkdir truffle-tutorial
cd truffle-tutorial
truffle init
```

정상적으로 구성이 완료되었다면 다음과 같은 폴더 구조를 갖게 됩니다.

![folder](https://i.imgur.com/xYkPBpj.png)

### 컴파일 및 컨트랙트 적용

여기서 `package.json` 파일을 만들고 아래 내용을 작성합니다.

```bash
{
"name": "erc721-contract",
"version": "1.0.0",
"description": "",
"main": "truffle-config.js",
"directories": {
"test": "test"
},
"scripts": {
"build": "truffle compile",
"start:dev": "truffle migrate --reset",
"start:testnet": "truffle migrate --network ropsten",
"start:mainnet": "truffle migrate --network main",
"flat": "truffle-flattener ./contracts/Token.sol > ./flat/Token_flat.sol",
"test": "echo \"Error: no test specified\" && exit 1"
},
"keywords": [],
"author": "",
"license": "ISC",
"dependencies": {
"@openzeppelin/contracts": "^4.3.3",
"@truffle/hdwallet-provider": "^1.7.0",
"eth-gas-reporter": "^0.2.22",
"ganache-cli": "^6.12.2",
"solium": "^1.2.5",
"truffle": "^5.4.21",
"truffle-flattener": "^1.5.0",
"truffle-hdwallet-provider": "^1.0.17",
"truffle-plugin-verify": "^0.5.18",
"web3": "^1.6.1"
},
"devDependencies": {
"@openzeppelin/test-environment": "^0.1.9",
"@openzeppelin/test-helpers": "^0.5.15",
"husky": "^4.2.3",
"lint-staged": "^10.5.3",
"solc": "^0.8.10",
"solidity-coverage": "^0.7.17"
}
}
```

위 `package.json` 은 `openzeppelin` 패키지 및 여러가지를 사용하기 위한 Dependencies를 포함하고 있습니다.

`package.json` 작성이 완료 되었다면 `yarn install` 로 설치를 진행합니다.

이제 실제 토큰 컨트랙트를 작성해야 합니다.

contracts/Token.sol 을 생성한 뒤 열고 아래 내용을 작성합니다.

```ts
pragma solidity ^0.8.0;

import "@openzeppelin/contracts/token/ERC721/presets/ERC721PresetMinterPauserAutoId.sol";

contract Token is 
    ERC721PresetMinterPauserAutoId
{
    constructor() ERC721PresetMinterPauserAutoId("TokenName", "TokenSymbol","baseTokenURI") {
        mint(msg.sender);
    }
}
```

위 코드는 `openzeppelin` 컨트랙트 내 ERC-721 프리셋 설정을 이용하여 토큰을 생성하는 컨트랙트 입니다.

위 코드에서 토큰이름, 심볼, URI를 적절히 변경하여 수정합니다.

이제 `truffle-config.js` 를 수정하여 어디에 배포할 것인지 설정을 작성해야합니다. 본 문서에서는 ropsten 테스트넷을 기준으로 작성하겠습니다.

`truffle-config.js`를 열고 아래와 같은 내용을 추가합니다.

```js
const HDWalletProvider = require('truffle-hdwallet-provider');

const fs = require('fs');
const mnemonic = fs.readFileSync(".secret").toString().trim();
const infuraKey = fs.readFileSync(".infuraKey").toString().trim();
...
...
networks: {
  ...
  ...
    ropsten: {
      provider: () => new HDWalletProvider(mnemonic, `https://ropsten.infura.io/v3/${infuraKey}`),
      network_id: 3,       // Ropsten's id
      gas: 5500000,        // Ropsten has a lower block limit than mainnet
      confirmations: 2,    // # of confs to wait between deployments. (default: 0)
      timeoutBlocks: 200,  // # of blocks before a deployment times out  (minimum/default: 50)
      skipDryRun: true     // Skip dry run before migrations? (default: false for public nets )
    },
   ...
   ...
}
```

위 설정에서 infura를 사용하고 있기 때문에 infura key와 이더리움 네트워크에 배포하기 위한 이더리움 주소의 Private key가 필요합니다. 이를 위해서 상단에 해당 키값을 불러올 수 있게 `fs.readFileSync()`를 추가하엿습니다

위의 설정을 모두 완료하셨다면 ```truffle migrate --network ropsten` 명령어를 입력하시면 컴파일 및 컨트랙트 트랜잭션 전송이 완료되며 실제 토큰이 생성됩니다.

## 마무리

이 글에서 `truffle`을 사용하여 ERC-721 토큰을 만드는 방법을 간단하게 소개하였습니다. 이 글에서는 미리 작성된 preset을 사용하여 토큰을 생성했지만 이러한 기능들을 커스텀하여 만드는것도 가능합니다. 본 문서에 작성된 내용을 구현한 깃허브 주소는 [링크](https://github.com/sweatpotato13/erc721-token-contract) 에 가시면 확인하실 수 있습니다.
