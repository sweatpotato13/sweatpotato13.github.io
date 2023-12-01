---
layout: post
title: "2023 Advent-of-spin Challenge 1"
date: "2023-12-01 08:05:44 +0900"
tags: [etc]
---

이 포스트는 [advent-of-spin](https://github.com/fermyon/advent-of-spin)에 업로드 된 Challenge 1에 대해서 진행했던 내용을 정리한 포스트 입니다.

Rust 언어로 진행하였으며 이 포스트에 게시된 소스코드는 [Github](https://github.com/sweatpotato13/advent-of-spin/tree/main/2023/CHALLENGE-1) 에서 확인할 수 있습니다.

## Spec

- `/index.html` 에 Static page 호스팅 하기 (크리스마스 관련된 무언가를 담아서)
- `/data` path에서 GET/POST method 구현하기
   - GET `/data?advent` : 저장된 value값 출력 / 성공시 200
   - POST: `/data?advent` body에 저장된 값 저장 / 성공시 201
        ```json
        {
            "value": "<Matt의 자동화된 위시리스트>"
        }
        ``` 
  - response header에 `Content-Type: application/json`


## Work

첫번째 요구사항인 `/index.html` 을 호스팅하기 위해 `static-fileserver` 을 사용합니다.

![spin new static-fileserver](https://i.imgur.com/L6O7rLf.png)

생성된 폴더 구조는 아래와 같습니다.

![folder static-fileserver](https://i.imgur.com/NqKwR8m.png)

assets 경로에 있는 파일들을 호스팅 해주는 파일서버 입니다. 

두번째 요구사항를 위해 key-value store를 만들어줍니다.

![spin new key-value](https://i.imgur.com/LcPpueb.png)

이 두 프로젝트를 별도의 서버가 아닌 하나의 서버로 작업할 것이기 때문에 두 프로젝트를 하나로 합칩니다.

합치고 난 이후에 이런 구조가 됩니다.

![folder key-value](https://i.imgur.com/18qYKKw.png)

- component는 key-value store인 `spin-key-value`와 `index.html` 을 호스팅해줄 `fileserver` 총 두가지 컴포넌트가 존재합니다.
- `/index.html` 경로에 호스팅을 해 주어야 하기 때문에 `fileserver` 의 route 는 `/` 로 하였습니다.
- 또한 `index.html` 만 사용하기 때문에 `/` 를 `/index.html` 로 변경하였습니다.
- spin-key-value 의 경우 `/data` 만이 존재하기 때문에 route를 `/data` 로 하였습니다.
- `key_value_stores = ["default"]` 를 `spin-key-value` 컴포넌트에 추가하였습니다.

이를 실행해보면 다음과 같이 route가 생성됩니다.

![route](https://i.imgur.com/QC70I7x.png)

이제 구현을 해야합니다.

`assets/index.html` 경로에 크리스마스 느낌의 이미지가 담긴 html 파일을 만들어줍니다.
        
```html
<!DOCTYPE html>
<html lang="en">
    <body>
        <img src="https://www.shutterstock.com/image-vector/flork-meme-christmas-vector-ilustration-600nw-2175960549.jpg" />
    </body>
</html>
```

![index.html](https://i.imgur.com/7oNx4zm.png)

`src/lib.rs` 에 GET, POST를 처리하기 위한 로직을 구현합니다.
```rust
use http::{Method, StatusCode};
use spin_sdk::{
    http::{IntoResponse, Response},
    http_component,
    key_value::Store,
};

#[http_component]
fn handle_request(req: http::Request<Vec<u8>>) -> anyhow::Result<impl IntoResponse> {
    let store = Store::open_default()?;

    let (status, body) = match *req.method() {
        Method::POST => {
            store.set(req.uri().path(), req.body().as_slice())?;
            println!(
                "Storing value in the KV store with {:?} as the key",
                req.uri().path()
            );
            (StatusCode::CREATED, None)
        }
        Method::GET => match store.get(req.uri().path())? {
            Some(value) => {
                println!("Found value for the key {:?}", req.uri().path());
                (StatusCode::OK, Some(value))
            }
            None => {
                println!("No value found for the key {:?}", req.uri().path());
                (StatusCode::NOT_FOUND, None)
            }
        },
        Method::DELETE => {
            store.delete(req.uri().path())?;
            println!("Delete key {:?}", req.uri().path());
            (StatusCode::OK, None)
        }
        Method::HEAD => {
            let code = if store.exists(req.uri().path())? {
                println!("{:?} key found", req.uri().path());
                StatusCode::OK
            } else {
                println!("{:?} key not found", req.uri().path());
                StatusCode::NOT_FOUND
            };
            (code, None)
        }
        _ => (StatusCode::METHOD_NOT_ALLOWED, None),
    };
    Ok(Response::builder()
        .status(status)
        .header("content-type", "application/json")
        .body(body)
        .build())
}
```

- `spin build` 로 빌드를 수행합니다.
    
    ![spin build](https://i.imgur.com/UbcktHT.png)
    
- `spin up` 으로 로컬에서 실행이 가능합니다.
    
    ![spin up](https://i.imgur.com/qz0yimw.png)
    
- curl을 사용하여 가볍게 테스트가 가능합니다.
    - POST
        
        ```bash
        curl -XPOST http://localhost:3000/data\?advent -H 'Content-Type: application/json' -d '{"value":"todolist
        ```
        
        ![POST](https://i.imgur.com/aw8LkSj.png)
        
    - GET
        
        ```bash
        curl -XGET http://localhost:3000/data\?advent -H 'Content-Type: application/json' -d '{"value":"todolist"}'
        ```
        
        ![GET](https://i.imgur.com/gmW30bS.png)
        
- 아래 명령어로 제출 전 테스트가 가능합니다.
    
    ```bash
    hurl --test test.hurl
    ```
    
    ![test](https://i.imgur.com/V2sHel8.png)