---
layout: post
title: "2024 Advent-of-spin Challenge 2"
date: "2024-12-15 08:05:44 +0900"
tags: [etc]
---

이 포스트는 [advent-of-spin](https://github.com/fermyon/advent-of-spin)에 업로드 된 Challenge 2에 대해서 진행했던 내용을 정리한 포스트 입니다.

Rust 언어로 진행하였으며 이 포스트에 게시된 소스코드는 [Github](https://github.com/sweatpotato13/advent-of-spin/tree/main/2024/CHALLENGE-2) 에서 확인할 수 있습니다.

## Spec

- Spin 3.0에서 추가된 `Component Dependencies` 기능을 사용하여 naughty-or-nice 점수 매기는 기능 구현하기
- `/naughty-or-nice.html` 에 특정 사람이름을 입력하고 점수를 확인할 수 있는 API가 추가된 프론트엔드 추가하기
- `/api/naughty-or-nice` path에서 GET method 구현하기
   - GET `/api/naughty-or-nice/:name` : name에 해당하는 naughty-or-nice 점수 출력
        ```json
        {
            "name": "John Doe",
            "score": 99
        }
        ``` 
  - response header에 `Content-Type: application/json`


## Work

이 Challenge도 Challenge 1과 마찬가지로 static page와 api가 구현된 wasm component를 사용하기 위해 challenge 1에서 만들어놓은 프로젝트를 기반으로 사용하려고 합니다.

우선 프론트엔드 부분을 구현하기 위해 `assets/naughty-or-nice.html` 파일을 생성합니다.

`assets/naughty-or-nice.html` 경로에 GET 요청을 보내면 아래와 같은 응답을 받을 수 있도록 구현합니다.
        
```html
<!DOCTYPE html>
<html>
<head>
  <title>Naughty or Nice</title>
</head>
<body>
  <h1>Naughty or Nice Checker</h1>
  <input type="text" id="nameInput" placeholder="Enter name">
  <button onclick="checkScore()">Check Score</button>
  <p id="result"></p>

  <script>
    async function checkScore() {
      const name = document.getElementById('nameInput').value;
      const response = await fetch(`/api/naughty-or-nice/${encodeURIComponent(name)}`);
      const data = await response.json();
      document.getElementById('result').textContent = `${data.name} is scored: ${data.score}`;
    }
  </script>
</body>
</html>
```

![index.html](https://i.imgur.com/IcqIna2.png)

이번에는 rust 코드를 작성하기 전에 `Component Dependencies` 기능을 사용하여 score를 출력하는 기능이 포함된 wasm component를 추가해보겠습니다.

우리는 이 컴포넌트를 구현하기 위해 [`componentize-js`](https://github.com/bytecodealliance/ComponentizeJS) 와 [`jco`](https://github.com/bytecodealliance/jco) 를 사용할 것입니다.

우선 이 두가지 패키지를 설치합니다

```bash
npm install -g @bytecodealliance/componentize-js @bytecodealliance/jco
```

우리는 JS/TS 코드를 wasm으로 빌드하기 위해 위 두가지 패키지를 사용하여 구현해 보겠습니다.

우선 js 코드를 작성합니다. `scoring.js` 파일을 생성하고 아래와 같이 작성합니다.

```js
/**
 * This module is the JS implementation of the `reverser` WIT world
 */

/**
 * The Javascript export below represents the export of the `reverse` interface,
 * which which contains `reverse-string` as it's primary exported function.
 */
export const scoring = {
    /**
     * This Javascript will be interpreted by `jco` and turned into a
     * WebAssembly binary with a single export (this `reverse` function).
     */
    scoring(name) {
        const randomFactor = Math.floor(Math.random() * 100);
        return Math.max(1, Math.min(randomFactor, 99));
    },
};
```

그리고 `scoring.wit` 파일을 생성하고 아래와 같이 작성합니다.

```wit
package local:scoring;

interface scoring {
    scoring: func(name: string) -> u32;
}

world component {
    export scoring;
}
```

위 두가지 파일을 이용하여 wasm 파일을 생성합니다.

```bash
jco componentize scoring.js -w scoring.wit -o scoring.component.wasm
```

![ss](https://i.imgur.com/9eBWDPR.png)

그렇게 되면 `scoring.component.wasm` 파일이 생성됩니다.

![ss](https://i.imgur.com/DJWVzMG.png)

이제 이 wasm 파일을 spin 프로젝트에 추가해야 합니다.

이를 위해서 `spin deps generate-bindings` 커맨드를 사용하여 wasm 파일에 대한 binding을 생성해 주어야 하는데 이를 위해서 spin plugin을 설치해 주어야 합니다.

- https://github.com/fermyon/spin-deps-plugin/tree/main?tab=readme-ov-file#installation

설치가 완료 되었다면 아래 명령어를 사용하여 binding을 생성합니다.

```bash
spin deps generate-bindings -L rust -o src/bindings -c challenge2
```

![ss](https://i.imgur.com/T7u1Zr3.png)

위 과정이 정상적으로 수행 되엇다면 `src/bindings` 경로에 파일이 생성된 것을 확인할 수 있습니다.

![ss](https://i.imgur.com/bMWyx3m.png)

![ss](https://i.imgur.com/DfrB6Kn.png)

이제 이 파일을 이용하여 rust 코드를 작성해보겠습니다.

`src/lib.rs` 에 GET을 처리하기 위한 로직을 구현합니다.
```rust
use serde::{Deserialize, Serialize};
use spin_sdk::http::Method;
use spin_sdk::http::{IntoResponse, Request, Response};
use spin_sdk::http_component;

mod bindings;

#[derive(Debug, Serialize, Deserialize)]
struct NaughtyOrNice {
    name: String,
    score: u32,
}

#[http_component]
fn handle_request(req: Request) -> anyhow::Result<impl IntoResponse> {
    println!("Received request: {:?}", req.method());

    let (status, body) = match *req.method() {
        Method::Get => {
            println!("GET request received");
            let path = req.uri();
            let parts: Vec<&str> = path.split('/').collect();
            let name = parts[5];
            let name = urlencoding::decode(name).unwrap();
            let score = bindings::deps::local::scoring::scoring::scoring(&name);
            println!("param: {:?}", name);
            let json_body = NaughtyOrNice {
                name: name.clone().to_string(),
                score,
            };
            (200, serde_json::to_vec(&json_body).unwrap())
        }
        _ => (404, Vec::new()),
    };

    Ok(Response::builder()
        .status(status)
        .header("content-type", "application/json")
        .body(body)
        .build())
}
```

- `spin build` 로 빌드를 수행합니다.
    
    ![spin build](https://i.imgur.com/1fo6oNc.png)
    
- `spin up` 으로 로컬에서 실행이 가능합니다.
    
    ![spin up](https://i.imgur.com/WEKZlkj.png)
    
- curl을 사용하여 가볍게 테스트가 가능합니다.
    - GET
        
        ```bash
        curl http://127.0.0.1:3000/api/naughty-or-nice/abc
        ```
        
        ![GET](https://i.imgur.com/my6j9y2.png)
        
- 아래 명령어로 제출 전 테스트가 가능합니다.
    
    ```bash
    hurl --test test.hurl
    ```
    
    ![test](https://i.imgur.com/PW8S1sJ.png)