---
layout: post
title: "2023 Advent-of-spin Challenge 4"
date: "2024-01-03 08:05:44 +0900"
tags: [etc]
---

이 포스트는 [advent-of-spin](https://github.com/fermyon/advent-of-spin)에 업로드 된 Challenge 4에 대해서 진행했던 내용을 정리한 포스트 입니다.

Rust 언어로 진행하였으며 이 포스트에 게시된 소스코드는 [Github](https://github.com/sweatpotato13/advent-of-spin/tree/main/2023/CHALLENGE-4) 에서 확인할 수 있습니다.

## Spec

- [Bulls'n'Cows](https://en.wikipedia.org/wiki/Bulls_and_Cows) 게임을 진행합니다. 숫자야구와 동일한 게임입니다.
- https://bulls-n-cows.fermyon.app/api API를 통해서 게임을 진행할 수 있으며 이 게임을 진행할 수 있는 로직을 구현합니다.
- https://bulls-n-cows.fermyon.app/api?guess=012 식으로 query에 guess 를 통해서 값을 전달할 수 있으며 응답은 아래와 같습니다.
- 이후 동일한 게임을 이어서 할 때 • `https://bulls-n-cows.fermyon.app/api?guess=012&id=1836633b-8dd1-41c2-b084-644b37c6fd1b` 처럼 id를 같이 전달합니다.

## Work

- 이번 챌린지는 로직 구현에 초점을 맞추어 진행합니다. 라우터는 `GET` 하나만 뚫어서 진행합니다.
- `spin new` 명령어를 이용하여 새로운 프로젝트를 생성해 줍니다.
    
    ![spin new](https://i.imgur.com/vuOsoAw.png)
    
- 그리고 생성된 프로젝트의 `Cargo.toml` 파일에 아래와 같이 dependencies를 추가해 줍니다.
    
    ![dependencies](https://i.imgur.com/aWJeD5Q.png)
    
    - request / response 객체를 serialize / deserialize 하기 위해 serde 라이브러리를 사용합니다.
- `src/lib.rs` 에  POST를 처리하기 위한 로직을 구현합니다.

    - 경우의 수가 많지 않아(0,1,2,3,4 만 사용) 모든 경우를 array에 넣고 모든 경우를 시도하는 방법으로 구현하였습니다.
    ```rust
    use http::StatusCode;
    use serde::{Deserialize, Serialize};
    use spin_sdk::{
        http::{IntoResponse, Method, Request, Response},
        http_component,
    };

    #[derive(Debug, Deserialize, Serialize)]
    struct GameResponse {
        cows: u64,
        bulls: u64,
        gameId: String,
        guesses: u64,
        solved: bool,
    }

    #[http_component]
    async fn handle_request(_req: http::Request<Vec<u8>>) -> anyhow::Result<impl IntoResponse> {
        let guess_numbers = [
            "013", "014", "021", "023", "024", "031", "032", "034", "102", "103", "104", "120",
            "123", "124", "130", "132", "134", "201", "203", "204", "210", "213", "214", "230", "231",
            "234", "301", "302", "304", "310", "312", "314", "320", "321", "324",
        ];

        let status = StatusCode::OK;
        let body = "Done".to_string();

        let uri = format!("https://bulls-n-cows.fermyon.app/api?guess=012");

        let req = Request::builder().method(Method::Get).uri(uri).build();

        // Send the request and await the response
        let res: Response = spin_sdk::http::send(req).await?;

        let response: GameResponse = serde_json::from_slice(res.body().to_vec().as_slice()).unwrap();     

        if response.solved {
            println!("{:?}", response);
            Ok(Response::builder()
            .status(status)
            .header("content-type", "application/json")
            .body(body)
            .build())
        }
        else {
            let game_id = response.gameId;
            for guess in guess_numbers.iter() {
                let uri = format!("https://bulls-n-cows.fermyon.app/api?guess={}&id={}", guess, game_id);
            
                let req = Request::builder().method(Method::Get).uri(uri).build();
            
                // Send the request and await the response
                let res: Response = spin_sdk::http::send(req).await?;
            
                let response: GameResponse = serde_json::from_slice(res.body().to_vec().as_slice()).unwrap();     
        
                if response.solved {
                    println!("{:?}", response);
                    break;
                }
            }
            Ok(Response::builder()
                .status(status)
                .header("content-type", "application/json")
                .body(body)
                .build())
        
            }
        }
    ```