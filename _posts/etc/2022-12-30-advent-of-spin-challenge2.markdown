---
layout: post
title: "Advent-of-spin Challenge 2"
date: "2022-12-30 08:04:44 +0900"
tags: [etc]
---

이 포스트는 [advent-of-spin](https://github.com/fermyon/advent-of-spin)에 업로드 된 Challenge 2에 대해서 진행했던 내용을 정리한 포스트 입니다.

Rust 언어로 진행하였으며 이 포스트에 게시된 소스코드는 [Github](https://github.com/sweatpotato13/advent-of-spin/tree/main/CHALLENGE-2) 에서 확인할 수 있습니다.

## Spec

- `/` 요청에 404 에러 띄우기
- `/lowercase` 요청에 아래와 같은 동작을 수행합니다.
  - Response Header에 `Content-Type: application/json`
  - POST Method 로 요청을 받고 body에 value 값을 포함하여 받습니다.
  - `{message: lowercase_string_of_value}` 를 응답으로 보냅니다.
- `/hello/*` 요청에 아래와 같은 동작을 수행합니다.
  - Response Header에 `Content-Type: application/json`
  - `{message: "Hello, world!}` 가 포함된 JSON Payload를 응답으로 보냅니다.
  - `/hello/cutewisp` 처럼 path가 뒤에 추가로 붙었을 경우 lower-case로 `Hello, cutewisp!`와 같이 응답을 보냅니다.
    - `outbount_http` 를 사용하여 위에서 만든 Endpoint로 요청을 보내서 구현하면 간단하게 구현할 수 있습니다.

## Work

- Spin 프레임워크가 하나의 파일에 2개 이상의 route를 포함하는걸 지원하지 않기 때문에 각각의 route에 대해서 별도의 파일로 구성해야 합니다.
- 이 Challenge 에서는 총 3개의 Route가 필요합니다 (`/` , `/lowercase` , `/hello/*` ) 그래서 아래와 같이 `spin.toml`을 설정해 3개의 디렉토리를 지정했습니다.

  - 각각의 디렉터리는 `spin new` 를 통해서 생성하였습니다.

  ```toml
  spin_version = "1"
  authors = ["Cute_Wisp <sweatpotato13@gmail.com>"]
  description = ""
  name = "challenge"
  trigger = { type = "http", base = "/" }
  version = "0.1.0"

  [[component]]
  id = "handle_404"
  source = "handle_404/target/wasm32-wasi/release/handle_404.wasm"
  allowed_http_hosts = ["insecure:allow-all"]
  [component.trigger]
  route = "/"
  [component.build]
  command = "cargo build --target wasm32-wasi --release"
  workdir = "handle_404"

  [[component]]
  id = "lowercase"
  source = "lowercase/target/wasm32-wasi/release/lowercase.wasm"
  allowed_http_hosts = ["insecure:allow-all"]
  [component.trigger]
  route = "/lowercase"
  [component.build]
  command = "cargo build --target wasm32-wasi --release"
  workdir = "lowercase"

  [[component]]
  id = "hello"
  source = "hello/target/wasm32-wasi/release/hello.wasm"
  allowed_http_hosts = ["insecure:allow-all"]
  [component.trigger]
  route = "/hello/..."
  [component.build]
  command = "cargo build --target wasm32-wasi --release"
  workdir = "hello"
  ```

- `handle_404` 컴포넌트는 단순하게 `/` 라우터를 통해서 접속하는 요청에 대해 404를 응답하는 코드를 작성하였습니다.

  ```rust
  use anyhow::Result;
  use spin_sdk::{
      http::{not_found, Request, Response},
      http_component,
  };

  /// A simple Spin HTTP component.
  #[http_component]
  fn handle_404(req: Request) -> Result<Response> {
      return not_found();
  }
  ```

- `lowercase` 컴포넌트는 아래와 같이 코드를 작성하였습니다.

  ```rust
  use anyhow::Result;
  use serde::{Deserialize, Serialize};
  use spin_sdk::{
      http::{not_found, Request, Response},
      http_component,
  };

  #[derive(Debug, Deserialize)]
  struct LowercaseRequest {
      value: String,
  }

  #[derive(Serialize)]
  struct LowercaseResponse {
      message: String,
  }

  /// A simple Spin HTTP component.
  #[http_component]
  fn lowercase(req: Request) -> Result<Response> {
      let method = req.method();
      if method == "POST" {
          let lowercase_request: LowercaseRequest =
              serde_json::from_slice(req.body().clone().unwrap().to_vec().as_slice()).unwrap();

          let lowercase_response = LowercaseResponse {
              message: lowercase_request.value.to_lowercase(),
          };

          let json_response = serde_json::to_string(&lowercase_response)?;

          return Ok(http::Response::builder()
              .status(200)
              .header("Content-Type", "application/json")
              .body(Some(json_response.into()))?);
      } else {
          return not_found();
      }
  }
  ```

  - `req.method()` 를 통해서 request 의 method를 가져올 수 있습니다.
  - `req.body()` 를 파싱하여 LowercaseRequest 형식의 JSON으로 파싱하여 lowercase_request 변수에 저장합니다. 이 과정에서 serde_json 패키지를 사용하였습니다.
  - `lowercase_request.value` 를 통해서 value 값을 가져와 이를 lowercase로 변환하고 response body에 담을 데이터를 정의합니다.
  - `json_response` 형식으로 만들어서 body에 담아 리턴합니다.
  - `.header("Content-Type", "application/json")` 도 추가 해 줍니다.

- `hello` 컴포넌트는 아래와 같이 코드를 작성하였습니다.

  ```rust
  use anyhow::Result;
  use serde::{Deserialize, Serialize};
  use spin_sdk::{
      http::{Request, Response},
      http_component,
  };

  #[derive(Serialize)]
  struct LowercaseRequest {
      value: String,
  }

  #[derive(Debug, Deserialize)]
  struct LowercaseResponse {
      message: String,
  }

  #[derive(Serialize)]
  struct HelloResponse {
      message: String,
  }

  /// A simple Spin HTTP component.
  #[http_component]
  fn hello(req: Request) -> Result<Response> {
      let default_value: &str = "world";

      let name: &str = req
          .headers()
          .get("spin-path-info")
          .unwrap()
          .to_str()
          .unwrap();

      let mut hello_response = HelloResponse {
          message: format!("Hello, {}!", default_value),
      };

      if name != "" {
          let lowercase_request = LowercaseRequest {
              value: name.strip_prefix("/").unwrap().to_string(),
          };
          let host: &str = req.headers().get("host").unwrap().to_str().unwrap();
          let spin_full_url: &str = req
              .headers()
              .get("spin-full-url")
              .unwrap()
              .to_str()
              .unwrap();

          let protocol = spin_full_url.split_once("://").unwrap().0;

          let uri = format!("{}://{}/lowercase", protocol, host);
          println!("{:?}", uri);
          let res = spin_sdk::outbound_http::send_request(
              http::Request::builder().method("POST").uri(uri).body(Some(
                  serde_json::to_string(&lowercase_request).unwrap().into(),
              ))?,
          )?;

          let lowercase_response: LowercaseResponse =
              serde_json::from_slice(res.body().clone().unwrap().to_vec().as_slice()).unwrap();

          hello_response.message = format!("Hello, {}!", lowercase_response.message);
      }

      let json_response = serde_json::to_string(&hello_response)?;

      Ok(http::Response::builder()
          .status(200)
          .header("Content-Type", "application/json")
          .body(Some(json_response.into()))?)
  }
  ```

  - path 가 없을 경우 기본 값을 `default_value` 로 정의하여 선언하였습니다.
  - header에 담긴 `spin-path-info` 를 참고하여 path를 가져옵니다.
  - `name` 이 빈 스트림이 아니라면 path가 있는것이므로 분기 처리를 합니다.
  - path가 없을경우 기본값을 사용하여 `json_response` 를 만들어 바로 리턴합니다.
  - path가 있을경우 path의 값을 따와서 `lowercase` route로 보낼 request를 정의합니다
  - 테스트 및 제출은 동일한 host에서 이루어지므로 해당 요청으로 들어온 host 및 protocol을 header에서 가져와 `lowercase` route의 host를 정의합니다.
  - `lowercase` route로 요청을 보내고 받은 응답을 가지고 `hello_response` 응답 메시지를 정의한 뒤 리턴합니다.

- `make test` 명령어로 테스트를 수행 해 보았습니다.
  ![Untitled](https://i.imgur.com/UjougUc.png)
- 아래 명령어로 과제 제출
  - `serviceUrl` 은 위에 deploy를 통해서 획득한 endpoint를 넣으면 됩니다.
  ```rust
  hurl --variable serviceUrl="https://challenge-rtbzeodc.fermyon.app" submit.hurl
  ```
  ![Untitled](https://i.imgur.com/LitQPFm.png)

---
