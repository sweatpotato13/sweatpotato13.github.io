---
layout: post
title: "spin SDK language에 따른 성능 차이 비교"
date: "2023-02-19 08:05:44 +0900"
tags: [etc]
---

이전에 [spin](https://blog.cutewisp.com/2022/12/28/spin/)이라고 하는 프레임워크에 대해서 소개해 드린적이 있었습니다.

Spin은 WebAssembly를 이용하여 Microservice를 구현할 수 있는 오픈소스 프레임워크인데 Rust, Go, Javascript/Typescript 등 많은 언어로 SDK가 제작되어 있습니다. 어떠한 언어를 사용하더라도 결국엔 wasm 형식으로 빌드웨어 실행하게 되는데 언어별로 성능에 차이가 있는지 궁금해 졌습니다.

관련해서 [discord](https://discord.gg/2gVFeDXs)에 글을 남겨봤는데 아래와 같은 답변을 얻었습니다.

![ss](https://i.imgur.com/nAEgUfV.png)

간단하게 JS/TS와 spin사이에 `quickjs-wasm` 이라고 하는 추가 계층이 존재해서 그 부분에서 성능차이가 날 수 있다고 합니다.

그래서 실제로 차이가 있는지 확인해 보기 위하여 실험을 해 보았습니다.

## Performance difference between Rust SDK and Typescript SDK

### Rust

Rust는 아래와 같은 코드를 사용하였습니다.

```rust
use anyhow::Result;
use spin_sdk::{
    http::{Request, Response},
    http_component,
};

/// A simple Spin HTTP component.
#[http_component]
fn handle_hello_rust(req: Request) -> Result<Response> {
    println!("{:?}", req.headers());
    Ok(http::Response::builder()
        .status(200)
        .header("foo", "bar")
        .body(Some("Hello, Fermyon".into()))?)
}
```

### Typescript

Typescript는 아래와 같은 코드를 사용하였습니다.

```typescript
import { HandleRequest, HttpRequest, HttpResponse } from "@fermyon/spin-sdk";

const encoder = new TextEncoder();

export const handleRequest: HandleRequest = async function (
  request: HttpRequest
): Promise<HttpResponse> {
  return {
    status: 200,
    headers: { foo: "bar" },
    body: encoder.encode("Hello from TS-SDK").buffer,
  };
};
```

### Benchmark

벤치마킹 툴로는 [plow](https://github.com/six-ddc/plow)를 사용하였으며 spin application은 로컬 기기인 M1 맥미니에서 구동하였습니다.

아래는 Rust SDK로 빌드한 spin application의 벤치마킹 결과입니다.

![rust-cli](https://i.imgur.com/kR90xF7.png)
![rust-web](https://i.imgur.com/CfCZh2q.png)

아래는 Typescript SDK로 빌드한 spin application의 벤치마킹 결과입니다.

![ts-cli](https://i.imgur.com/nwofoOm.png)
![ts-web](https://i.imgur.com/SNrvYV7.png)

결과적으로 Latency나 RPS(Request Per Second)와 같은 수치가 Rust SDK로 빌드한 spin application이 Typescript SDK를 사용한 그것보다 약 2배 가량의 성능의 차이가 있었습니다.

간단한 Hello world만 출력하는 API에 대해서도 이정도의 차이를 보이게 된다면 더 복잡한 프로젝트에 대해서는 더더욱 차이가 벌어지지 않을까 생각 합니다.
