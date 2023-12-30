---
layout: post
title: "2023 Advent-of-spin Challenge 2"
date: "2023-12-29 08:05:44 +0900"
tags: [etc]
---

이 포스트는 [advent-of-spin](https://github.com/fermyon/advent-of-spin)에 업로드 된 Challenge 2에 대해서 진행했던 내용을 정리한 포스트 입니다.

Rust 언어로 진행하였으며 이 포스트에 게시된 소스코드는 [Github](https://github.com/sweatpotato13/advent-of-spin/tree/main/2023/CHALLENGE-2) 에서 확인할 수 있습니다.

## Spec

- `/` 에 `POST` method 구현하기
   - POST: `/` body로 아래의 형식대로 값을 전달
        ```json
        {
            "kids": [1, 4 , 5, 6, 3, 2, 7],
            "weight": [23, 45, 23, 43, 12, 32, 45],
            "capacity": 120
        }
        ``` 
  - 결과 값으로 제공된 데이터 세트에 대해서 얼마나 많은 Kids에게 선물을 줄 수 있는지 출력
        ```json
        {
            "kids": 18
        }
        ```
  - response header에 `Content-Type: application/json`


## Work

- 이번 챌린지는 단순하게 POST 메소드로 라우팅을 하나만 뚫으면 되는 간단한 챌린지 입니다.
- `spin new` 명령어를 이용하여 새로운 프로젝트를 생성해 줍니다.
    
    ![spin new](https://i.imgur.com/vuOsoAw.png)
    
- 그리고 생성된 프로젝트의 `Cargo.toml` 파일에 아래와 같이 dependencies를 추가해 줍니다.
    
    ![dependencies](https://i.imgur.com/aWJeD5Q.png)
    
    - request / response 객체를 serialize / deserialize 하기 위해 serde 라이브러리를 사용합니다.
- 이제 구현을 해야합니다.
- 문제를 해결하기 위해서 DP를 사용하여 로직을 구현할 수 있습니다.
    
    ```rust
    fn optimal_kids_reached(kids: Vec<usize>, weight: Vec<usize>, capacity: usize) -> usize {
        let n = kids.len();
        let mut dp = vec![vec![0; capacity + 1]; n + 1];
    
        for i in 1..=n {
            for w in 1..=capacity {
                if weight[i - 1] <= w {
                    dp[i][w] = dp[i - 1][w].max(kids[i - 1] + dp[i - 1][w - weight[i - 1]]);
                } else {
                    dp[i][w] = dp[i - 1][w];
                }
            }
        }
    
        dp[n][capacity]
    }
    ```
    
- `src/lib.rs` 에  POST를 처리하기 위한 로직을 구현합니다.
    
    ```rust
    use http::{Method, StatusCode};
    use serde::{Deserialize, Serialize};
    use spin_sdk::{
        http::{IntoResponse, Response},
        http_component,
    };
    
    #[derive(Debug, Deserialize, Serialize)]
    struct KidsRequest {
        kids: Vec<usize>,
        weight: Vec<usize>,
        capacity: usize,
    }
    
    #[derive(Debug, Deserialize, Serialize)]
    struct KidsResponse {
        kids: usize,
    }
    
    fn optimal_kids_reached(kids: Vec<usize>, weight: Vec<usize>, capacity: usize) -> usize {
        let n = kids.len();
        let mut dp = vec![vec![0; capacity + 1]; n + 1];
    
        for i in 1..=n {
            for w in 1..=capacity {
                if weight[i - 1] <= w {
                    dp[i][w] = dp[i - 1][w].max(kids[i - 1] + dp[i - 1][w - weight[i - 1]]);
                } else {
                    dp[i][w] = dp[i - 1][w];
                }
            }
        }
    
        dp[n][capacity]
    }
    
    #[http_component]
    fn handle_request(req: http::Request<Vec<u8>>) -> anyhow::Result<impl IntoResponse> {
        let (status, body) = match *req.method() {
            Method::POST => {
                let request: KidsRequest =
                    serde_json::from_slice(req.body().clone().to_vec().as_slice()).unwrap();
    
                let kids = request.kids;
                let weight = request.weight;
                let capacity = request.capacity;
    
                let maximum_kids = optimal_kids_reached(kids, weight, capacity);
    
                let response = KidsResponse { kids: maximum_kids };
    
                let json_response = serde_json::to_string(&response)?;
    
                (StatusCode::OK, json_response)
            }
            _ => (StatusCode::METHOD_NOT_ALLOWED, String::new()),
        };
        Ok(Response::builder()
            .status(status)
            .header("content-type", "application/json")
            .body(body)
            .build())
    }
    ```
    
- `spin build` 로 빌드를 수행합니다.
    
    ![spin build](https://i.imgur.com/KvZQn6T.png)
    
- `spin up` 으로 로컬에서 실행이 가능합니다.
    
    ![spin up](https://i.imgur.com/0mSQsgN.png)
    
- curl을 사용하여 가볍게 테스트가 가능합니다.
    
    ```bash
    curl localhost:3000 -H 'Content-Type: application/json' -d '{"kids":[1,4,5,6,3,2,7], "weight":[23,45,23,43,12,32,45], "capacity":120}'
    ```
    
    ![curl](https://i.imgur.com/f8xD60f.png)
    
- 아래 명령어로 제출 전 테스트가 가능합니다.
    
    ```bash
    hurl --test test.hurl
    ```
    
    ![test](https://i.imgur.com/VcyvuRO.png)