---
layout: post
title: "2024 Advent-of-spin Challenge 4"
date: "2024-12-26 08:05:44 +0900"
tags: [etc]
---

이 포스트는 [advent-of-spin](https://github.com/fermyon/advent-of-spin)에 업로드 된 Challenge 4에 대해서 진행했던 내용을 정리한 포스트 입니다.

Rust 언어로 진행하였으며 이 포스트에 게시된 소스코드는 [Github](https://github.com/sweatpotato13/advent-of-spin/tree/main/2024/CHALLENGE-4) 에서 확인할 수 있습니다.

## Treasure hunt

실제 challenge의 내용은 보물찾기를 통해 찾아야 합니다.

시작점은 다음 엔드포인트입니다

```
curl https://treasurehunt.fermyon.app/start
```

![image.png](https://i.imgur.com/eokV20v.png)

단서를 토대로 아래 명령어를 통해서 POST 요청을 보냈습니다.

```bash
curl -X POST https://treasurehunt.fermyon.app/artist-check \
-H "Content-Type: application/json" \
-d '{"name": "Bryan Adams"}'
```

![image.png](https://i.imgur.com/zFWnHvd.png)

다음 힌트를 얻기 위해 `GET /song-hint` 엔드포인트로 요청을 보냅니다

```
curl https://treasurehunt.fermyon.app/song-hint
```

![image.png](https://i.imgur.com/LIaiCbD.png)

이 base64로 인코딩된 텍스트를 디코딩해보면 노래 가사를 알 수 있을 것 같습니다. 터미널에서 다음 명령어로 디코딩해봅니다.

```
echo "U2FpZCBTYW50YSB0byBhIGJveSBjaGlsZCAiV2hhdCBoYXZlIHlvdSBiZWVuIGxvbmdpbmcgZm9yPyIKIkFsbCBJIHdhbnQgZm9yIENocmlzdG1hcyBpcyBhIFJvY2sgYW5kIFJvbGwgZWxlY3RyaWMgZ3VpdGFyIgpBbmQgYXdheSB3ZW50IFJ1ZG9scGggYSB3aGl6emluZyBsaWtlIGEgc2hvb3Rpbmcgc3Rhci4K" | base64 -d
```

![image.png](https://i.imgur.com/ffalzq2.png)

이 가사는 Run Rudolph Run의 일부입니다.

아래 명령어로 답을 제출해봅니다.

```bash
curl -X POST https://treasurehunt.fermyon.app/song-check \
-H "Content-Type: application/json" \
-d '{"name": "Run Rudolph Run"}'
```

![image.png](https://i.imgur.com/hZ3oQrv.png)

아래 명령어로 마지막 문제를 확인합니다.

```
curl https://treasurehunt.fermyon.app/final-riddle
```

![image.png](https://i.imgur.com/6jHZYoD.png)

아래 명령어로 정답을 확인합니다.

```
curl -X POST https://treasurehunt.fermyon.app/answer \
-H "Content-Type: application/json" \
-d '{"name": "Dasher"}'
```

![image.png](https://i.imgur.com/epp3YF0.png)

이제 실제 구현 spec이 나왔습니다.

## Spec

- `GET /api/my` 엔드포인트 생성
    - `content-type` 헤더는 `application/json` 이어야 함
    - 아래 포맷을 만족해야함
        
        ```json
        {
          "favorite" : {
            "music": "YOUR_LINK_HERE"
          }
        } 
        ```
        

## Work

- 이번 챌린지는 단순하게 `GET` 메소드로 라우팅을 하나만 뚫으면 되는 간단한 챌린지 입니다.
- `spin new` 명령어를 이용하여 새로운 프로젝트를 생성해 줍니다.
    
    ![image.png](https://i.imgur.com/2HnvU8k.png)
    
- `/api/my` 라우팅을 지정하기 위해 `spin.toml` 을 수정합니다.
    
    ![image.png](https://i.imgur.com/vLbw0f1.png)
    
    - `trigger.http` 부분의 route를
    
    ```toml
    route = "/api/my"
    ```
    
    로 수정 해 주었습니다.
    
- `src/lib.rs` 를 구현하기 위해 아래처럼 코드를 작성합니다.
    
    ```rust
    use spin_sdk::http::{IntoResponse, Method, Request, Response};
    use spin_sdk::http_component;
    
    #[http_component]
    fn handle_request(req: Request) -> anyhow::Result<impl IntoResponse> {
        println!("Received request: {:?}", req.method());
    
        let (status, body) = match *req.method() {
            Method::Get => {
                let json_body = serde_json::json!({
                    "favorite": {
                        "music": "https://music.apple.com/de/playlist/advent-of-spin-2024/pl.u-xky2TDpLaZ?l=en-GB"
                    }
                });
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
    
    - serde_json 을 dependency로 사용하기 때문에 `cargo add serde_json` 을 수행하여 의존성 추가를 해 주어야 합니다.

# Test

- `hurl --test test.hurl` 명령어로 테스트가 가능합니다.
    
    ![image.png](https://i.imgur.com/kuDdkTi.png)