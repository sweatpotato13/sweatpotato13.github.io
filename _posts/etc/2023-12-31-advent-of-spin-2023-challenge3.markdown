---
layout: post
title: "2023 Advent-of-spin Challenge 3"
date: "2023-12-31 08:05:44 +0900"
tags: [etc]
---

이 포스트는 [advent-of-spin](https://github.com/fermyon/advent-of-spin)에 업로드 된 Challenge 3에 대해서 진행했던 내용을 정리한 포스트 입니다.

Rust 언어로 진행하였으며 이 포스트에 게시된 소스코드는 [Github](https://github.com/sweatpotato13/advent-of-spin/tree/main/2023/CHALLENGE-3) 에서 확인할 수 있습니다.

## Spec

- `/` 에 `POST` method 구현하기
   - POST: `/` body로 아래의 형식대로 값을 전달
        ```json
        {
            "place": "North Pole",
            "characters": ["Santa Claus", "The Grinch", "a pingvin"],
            "objects": ["A spoon", "Two presents", "Palm tree"]
        }
        ``` 
  - 결과 값으로 제공된 데이터 세트에 대해서 LLM을 사용하여 크리스마스와 관련된 이야기를 만들어 출력
        ```json
        {
            "story": "<YOUR STORY HERE>"
        }
        ```
  - 상태코드 200으로 반환 
  - response header에 `Content-Type: application/json`


## Work

- 이번 챌린지는 단순하게 POST 메소드로 라우팅을 하나만 뚫으면 되는 간단한 챌린지 입니다.
- `spin new` 명령어를 이용하여 새로운 프로젝트를 생성해 줍니다.
    
    ![spin new](https://i.imgur.com/vuOsoAw.png)
    
- 그리고 생성된 프로젝트의 `Cargo.toml` 파일에 아래와 같이 dependencies를 추가해 줍니다.
    
    ![dependencies](https://i.imgur.com/aWJeD5Q.png)
    
    - request / response 객체를 serialize / deserialize 하기 위해 serde 라이브러리를 사용합니다.
- 이번 챌린지에선 LLM을 사용하므로 관련 설정을 spin.toml 에 추가합니다.
    ```toml
        [component.challenge]
        source = "target/wasm32-wasi/release/challenge.wasm"
        allowed_outbound_hosts = []
        ai_models = ["llama2-chat"]
        [component.challenge.build]
        command = "cargo build --target wasm32-wasi --release"
        watch = ["src/**/*.rs", "Cargo.toml"]
    ```
- `src/lib.rs` 에  POST를 처리하기 위한 로직을 구현합니다.
    
    ```rust
    use http::{Method, StatusCode};
    use serde::{Deserialize, Serialize};
    use spin_sdk::{
        http::{IntoResponse, Response},
        http_component, llm,
    };

    #[derive(Debug, Deserialize, Serialize)]
    struct StoryRequest {
        place: String,
        characters: Vec<String>,
        objects: Vec<String>,
    }

    #[derive(Debug, Deserialize, Serialize)]
    struct StoryResponse {
        story: String,
    }

    fn make_story(place: String, characters: Vec<String>, objects: Vec<String>) -> String {
        let model = llm::InferencingModel::Llama2Chat;
        let prompt = format!(
            "Can you make story with place {:?} and these characters {:?} and this objects {:?}",
            place, characters, objects
        );
        let inference = llm::infer(model, &prompt);
        format!("{:?}", inference)
    }

    #[http_component]
    fn handle_request(req: http::Request<Vec<u8>>) -> anyhow::Result<impl IntoResponse> {
        let (status, body) = match *req.method() {
            Method::POST => {
                let request: StoryRequest =
                    serde_json::from_slice(req.body().clone().to_vec().as_slice()).unwrap();

                let place = request.place;
                let characters = request.characters;
                let objects = request.objects;

                let story = make_story(place, characters, objects);

                let response = StoryResponse { story: story };

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