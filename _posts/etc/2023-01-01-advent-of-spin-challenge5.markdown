---
layout: post
title: "Advent-of-spin Challenge 5"
date: "2023-01-01 08:05:44 +0900"
tags: [etc]
---

이 포스트는 [advent-of-spin](https://github.com/fermyon/advent-of-spin)에 업로드 된 Challenge 4에 대해서 진행했던 내용을 정리한 포스트 입니다.

Rust 언어로 진행하였으며 이 포스트에 게시된 소스코드는 [Github](https://github.com/sweatpotato13/advent-of-spin/tree/main/CHALLENGE-5) 에서 확인할 수 있습니다.

## Spec

- `/${key}` route에 PUT 요청을 받아 String 값을 DB에 저장합니다.
  - Request body는 JSON 형식의 `{"value": "somedata'}` 형식이어야 합니다.
- `/${key}` route에 GET 요청을 받아 DB에 저장된 값을 불러옵니다.
  - Response는 JSON 형식의 `{"value": "somedata'}` 형식이어야 합니다.

## Work

- DB 읽기 / 쓰기 기능이 들어가야 하므로 이 글에서는 Postgres를 로컬에 구축하여 사용하였습니다.
  ![Untitled](https://i.imgur.com/mpReDxm.png)
- dev 테이블을 만들어 key, value로 column을 구성하였습니다.
  ![Untitled](https://i.imgur.com/4gE6XgG.png)
- `spin.toml` 에 DB 접속 정보를 추가해줍니다.

  ```toml
  spin_version = "1"
  authors = ["Cute_Wisp <sweatpotato13@gmail.com>"]
  description = ""
  name = "challenge"
  trigger = { type = "http", base = "/" }
  version = "0.1.0"

  [[component]]
  environment = { DB_URL = "host=localhost user=postgres password=postgres dbname=postgres" }
  id = "challenge"
  source = "target/wasm32-wasi/release/challenge.wasm"
  [component.trigger]
  route = "/..."
  [component.build]
  command = "cargo build --target wasm32-wasi --release"
  ```

- `src/lib.rs` 를 아래와 같이 작성합니다.

  ```rust
  use anyhow::Result;
  use http::Method;
  use serde::{Deserialize, Serialize};
  use spin_sdk::{
      http::{not_found, Request, Response},
      http_component,
      pg::{self, Decode},
  };

  const DB_URL_ENV: &str = "DB_URL";

  #[derive(Debug, Deserialize, Serialize, Clone)]
  struct Data {
      value: String,
  }

  #[derive(Debug, Deserialize, Serialize, Clone)]
  struct DataInt {
      value: i32,
  }

  /// A simple Spin HTTP component.
  #[http_component]
  fn challenge(req: Request) -> Result<Response> {
      match req.method().to_owned() {
          Method::GET => read(req),
          Method::PUT => write(req),
          _ => not_found(),
      }
  }

  fn read(_req: Request) -> Result<Response> {
      let key: &str = _req
          .headers()
          .get("spin-path-info")
          .unwrap()
          .to_str()
          .unwrap()
          .strip_prefix('/')
          .unwrap();

      let mut value_response = DataInt { value: 0 };

      let address = std::env::var(DB_URL_ENV)?;

      let sql = format!("SELECT value FROM dev WHERE key='{}'", key);
      let rowset = pg::query(&address, &sql[..], &[])?;
      for row in rowset.rows {
          let value = i32::decode(&row[0])?;
          value_response.value = value;
      }

      let json_response = serde_json::to_string(&value_response)?;

      return Ok(http::Response::builder()
          .status(200)
          .header("Content-Type", "application/json")
          .body(Some(json_response.into()))?);
  }

  fn write(_req: Request) -> Result<Response> {
      let key: &str = _req
          .headers()
          .get("spin-path-info")
          .unwrap()
          .to_str()
          .unwrap()
          .strip_prefix('/')
          .unwrap();

      let value_request: Data =
          serde_json::from_slice(_req.body().clone().unwrap().to_vec().as_slice()).unwrap();

      let address = std::env::var(DB_URL_ENV)?;

      let sql = format!(
          "INSERT INTO dev (key, value) VALUES ('{}', '{}')",
          key, value_request.value
      );
      let executed = pg::execute(&address, &sql[..], &[])?;

      return Ok(http::Response::builder()
          .status(200)
          .header("Content-Type", "application/json")
          .body(Some("done".into()))?);
  }
  ```

- `make test` 로 테스트 수행
  ![Untitled](https://i.imgur.com/BrFx7j6.png)

---
