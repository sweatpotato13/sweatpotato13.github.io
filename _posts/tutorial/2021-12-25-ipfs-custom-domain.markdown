---
layout: post
title: "IPFS 서버에 커스텀 도메인 주소 사용하기"
date: "2021-12-25 13:03:44 +0900"
tags: [tutorial, blockchain]
---

![Untitled](https://encrypted-tbn0.gstatic.com/images?q=tbn:ANd9GcTtnOVLNcFMBoIoyca7IyYa866nbko7WXQlzg&usqp=CAU)

## About

IPFS를 사용하여 파일 및 폴더를 저장하게 되면 `https://ipfs.io/ipfs/<CID>` 형식의 주소로 데이터가 저장된다. 하지만 CID가 Hash 기반으로 정해지기 때문에 폴더의 경우 폴더 내 데이터가 변경된다면 폴더의 주소 또한 바뀌게 된다. 

하지만 데이터가 변경되더라도 고정된 도메인을 사용해야할 경우가 있는데 이런 경우 커스텀 도메인을 적용하여 URL을 고정시킬 수 있다.

본 문서에서는 어떻게 커스텀 도메인을 적용하여 IPFS를 사용할 수 있는지에 대한 방법을 작성하였다.

### IPNS

https://docs.ipfs.io/concepts/ipns/

IPFS는 콘텐츠 기반 주소 지정을 사용 합니다. 다른 사람 과 같은 IPFS 주소를 공유하려는 경우 콘텐츠를 업데이트할 때마다 해당 사람에게 새 링크를 제공해야 합니다.

IPNS(InterPlanetary Name System)는 업데이트할 수 있는 주소를 만들어 이 문제를 해결합니다.

다음과 같은 폴더가 IPFS에 업로드 되어 있습니다.

![ipfs-folder](https://i.imgur.com/szu1Dom.png)

위 사진의 폴더에 접근하기 위해서는 `https://ipfs.io/ipfs/QmYTqQ2XeJ3XyXf4BGWFYm4o87qGXHgG2MMETSk7PUMZsQ` 링크에 접근하여야 하지만 폴더 내의 컨텐츠가 변경된다면 위 주소도 바뀌게 되어 위 주소를 사용하는 사용자들에게 새로운 주소를 알려주어야 합니다.

![s](https://i.imgur.com/1tjE2in.png)

이를 고정된 도메인으로 사용하기 위해서 도메인의 DNS를 변경하여 위 링크를 나의 소유의 도메인으로 접근할 수 있도록 만들어 줍니다.

![dns](https://i.imgur.com/jHY86fI.png)

![dns](https://i.imgur.com/UefKEgG.png)

위 사진과 같이 CNAME과 TXT DNS를 설정해 줍니다.

```txt
TXT:
_dnslink.<subdomain>

dnslink=/ipfs/<CID>
----
CNAME:
<subdomain>

gateway.ipfs.io.
```

위 DNS가 적용이 되었다면 `https://ipfs.io/ipfs/QmYTqQ2XeJ3XyXf4BGWFYm4o87qGXHgG2MMETSk7PUMZsQ` 링크로 접근 가능하던 IPFS내 폴더가 `https://ipfs.io/ipns/ipfs-image.cutewisp.com`위 주소로도 접근 가능해집니다.

![ss](https://i.imgur.com/ww324qZ.png)

이후에 컨텐츠가 변경되어 CID가 바뀌는 경우 DNS 설정으로 다시 돌아가 CID만 변경하여 입력해 주면 동일한 도메인으로 해당 데이터를 접근할 수 있게 됩니다.

## 마무리

이 글에서 ipns를 사용하여 ipfs에 도메인을 적용하여 고정된 URL을 만드는 방법을 소개하였습니다. NFT등을 사용할때 주로 ipfs를 사용하게 되는데 NFT의 컨텐츠를 추가할 경우 컨텐츠가 바뀌어 baseURL이 변경되는 경우가 있을것이다. 이럴 때 baseURL자체를 변경해도 되지만 이런식으록 고정된 도메인을 사용하는 방법도 좋을 것이다.