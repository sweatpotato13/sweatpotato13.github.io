---
layout: post
title: "Advent-of-spin Challenge 1"
date: "2022-12-30 08:03:44 +0900"
tags: [etc]
---

# Advent-of-spin Challenge 1

이 포스트는 [advent-of-spin](https://github.com/fermyon/advent-of-spin)에 업로드 된 Challenge 1에 대해서 진행했던 내용을 정리한 포스트 입니다.

Rust 언어로 진행하였으며 이 포스트에 게시된 소스코드는 [Github](https://github.com/sweatpotato13/advent-of-spin/tree/main/CHALLENGE-1) 에서 확인할 수 있습니다.

## Spec

- Response의 Header에 `Content-Type: application/json` 명시하기
- `{message: "Hello, world!"}` 형식의 Json Payload를 Response에 담기

## Work

- `spin new` 명령어로 새로운 프로젝트를 생성합니다.
  ![Untitled](https://i.imgur.com/ks0Oxwn.png)
  ![Untitled](https://i.imgur.com/Bm6wQyo.png)
- `spin build` 로 빌드 할 수 있습니다
  ![Untitled](https://i.imgur.com/Pz65ZcE.png)
- `spin up` 으로 실행 가능
  ![Untitled](https://i.imgur.com/XbzVSuu.png)
  ![Untitled](https://i.imgur.com/IhdUJsA.png)
- `lib.rs` 파일을 아래와 같이 수정하여 response body의 데이터를 변경해 주었습니다.
  ```rust
  use anyhow::Result;
  use spin_sdk::{
      http::{Request, Response},
      http_component,
  };

  /// A simple Spin HTTP component.
  #[http_component]
  fn challenge(req: Request) -> Result<Response> {
      println!("{:?}", req.headers());
      let data = r#"{
          "message": "Hello, world!"
      }"#;
      Ok(http::Response::builder()
          .status(200)
          .header("Content-Type", "application/json")
          .body(Some(data.into()))?)
  }
  ```
- 위에서 안내한 대로 빌드 후 서버를 실행해 Response를 확인 해 보았습니다.
  - 정상적으로 header에 `application/json` 이 명시되어 있고 response body도 정상적으로 출력 되고 있음을 확인할 수 있습니다.
  ![Untitled](https://i.imgur.com/jNOb9tj.png)
- `make test` 명령어로 테스트를 수행 해 보았습니다.
  - [Hurl](https://hurl.dev/) 을 설치해야 합니다.
    ![Untitled](https://i.imgur.com/RRudjDL.png)
- `spin deploy` 로 cloud 에 deploy 합니다.
  ![Untitled](https://i.imgur.com/VmLHTpE.png)
- 아래 명령어로 과제 제출
  - `serviceUrl` 은 위에 deploy를 통해서 획득한 endpoint를 넣으면 됩니다.
  ```rust
  hurl --variable serviceUrl="https://challenge-rtbzeodc.fermyon.app" submit.hurl
  ```
  ![Untitled](https://i.imgur.com/QaLN3ge.png)

---
