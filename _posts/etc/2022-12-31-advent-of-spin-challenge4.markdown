---
layout: post
title: "Advent-of-spin Challenge 4"
date: "2022-12-31 08:05:44 +0900"
tags: [etc]
---

이 포스트는 [advent-of-spin](https://github.com/fermyon/advent-of-spin)에 업로드 된 Challenge 4에 대해서 진행했던 내용을 정리한 포스트 입니다.

Rust 언어로 진행하였으며 이 포스트에 게시된 소스코드는 [Github](https://github.com/sweatpotato13/advent-of-spin/tree/main/2023/CHALLENGE-4) 에서 확인할 수 있습니다.

## Spec

- `/distance-latlong` route에서 아래와 같은 데이터 형식을 포함한 요청을 POST로 받아야 합니다.
  - Payload body: `{ d1: { lat: 0f, long: 0f }, d2: { lat: 0f, long: 0f }}`
- Response Header에 `Content-Type: application/json` 를 포함하여 아래와 같은 데이터 형식을 반환합니다.
  - Payload response body: `{ distance: 0f }`\*\*\*\*

## Work

- `spin new`로 새로운 프로젝트를 만들고 `spin.toml` 에서 `/disatance-latlong` 에 대해서 요청을 받을 수 있게 route를 수정합니다.

  ```toml
  spin_version = "1"
  authors = ["Cute_Wisp <sweatpotato13@gmail.com>"]
  description = ""
  name = "challenge"
  trigger = { type = "http", base = "/" }
  version = "0.1.0"

  [[component]]
  id = "challenge"
  source = "target/wasm32-wasi/release/challenge.wasm"
  [component.trigger]
  route = "/distance-latlong"
  [component.build]
  command = "cargo build --target wasm32-wasi --release"
  ```

- `src/lib.rs` 파일을 아래와 같이 수정하였습니다.

  - 두 지점 사이의 거리 계산을 위해 [Haversine formula](https://crates.io/crates/haversine) 사용
  - 마일 → 해리 (nautical miles) 변환을 위해서 [unit-conversions](https://crates.io/crates/unit-conversions) 사용

  ```rust
  extern crate haversine;
  use anyhow::Result;
  use http::*;
  use serde::{Deserialize, Serialize};
  use spin_sdk::{
      http::{Request, Response},
      http_component,
  };

  #[derive(Debug, Serialize, Deserialize)]
  pub struct Coordinate {
      lat: f64,
      long: f64,
  }

  #[derive(Debug, Serialize, Deserialize)]
  pub struct DistanceRequest {
      d1: Coordinate,
      d2: Coordinate,
  }

  #[derive(Debug, Serialize)]
  pub struct DistanceResponse {
      distance: f64,
  }

  /// A simple Spin HTTP component.
  #[http_component]
  fn distance_latlog(req: Request) -> Result<Response> {
      assert_eq!(*req.method(), Method::POST);

      let distance_request: DistanceRequest =
          serde_json::from_slice(req.body().clone().unwrap().to_vec().as_slice()).unwrap();

      let d1: Coordinate = distance_request.d1;
      let d2: Coordinate = distance_request.d2;
      let start1 = haversine::Location {
          latitude: d1.lat,
          longitude: d1.long,
      };
      let end1 = haversine::Location {
          latitude: d2.lat,
          longitude: d2.long,
      };
      let distance = haversine::distance(start1, end1, haversine::Units::Miles);

      let nautical_miles = unit_conversions::length::miles::to_nautical_miles(distance);

      let rounded_miles = (nautical_miles * 10.0).round() / 10.0;

      let distance_response = DistanceResponse {
          distance: rounded_miles,
      };

      let json_response = serde_json::to_string(&distance_response)?;

      Ok(http::Response::builder()
          .status(200)
          .header("Content-Type", "application/json")
          .body(Some(json_response.into()))?)
  }
  ```

- `make test` 로 테스트 수행
  ![Untitled](https://i.imgur.com/4BunoGD.png)
- `spin deploy`로 클라우드에 업로드
  ![Untitled](https://i.imgur.com/l3BWwO3.png)
- 아래 명령어로 Submit
  - `serviceUrl` 에는 위에서 얻은 클라우드 주소를 넣습니다.
  ```rust
  hurl --variable serviceUrl="https://challenge-rcunxh7u.fermyon.app" submit.hurl
  ```
  ![Untitled](https://i.imgur.com/DCr458R.png)

---
