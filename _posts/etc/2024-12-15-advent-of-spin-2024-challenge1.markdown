---
layout: post
title: "2024 Advent-of-spin Challenge 1"
date: "2024-12-15 08:05:44 +0900"
tags: [etc]
---

이 포스트는 [advent-of-spin](https://github.com/fermyon/advent-of-spin)에 업로드 된 Challenge 1에 대해서 진행했던 내용을 정리한 포스트 입니다.

Rust 언어로 진행하였으며 이 포스트에 게시된 소스코드는 [Github](https://github.com/sweatpotato13/advent-of-spin/tree/main/2024/CHALLENGE-1) 에서 확인할 수 있습니다.

## Spec

- `/index.html` 에 Wishlist Management Frontend 호스팅 하기
- `/api/wishlists` path에서 GET/POST method 구현하기
   - GET `/api/wishlists` : 저장된 모든 wishlists 한번에 출력
   - POST: `/api/wishlists` 새로운 wishlists 저장
        ```json
        {
            "name": "John Doe",
            "items": [
                "Ugly Sweater",
                "Gingerbread House",
                "Stanley Cup Winter Edition"
            ]
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

`assets/index.html` 경로에 GET 요청을 보내면 아래와 같은 응답을 받을 수 있도록 구현합니다.
        
```html
<!DOCTYPE html>
<html lang="en">
<head>
  <meta charset="UTF-8">
  <meta name="viewport" content="width=device-width, initial-scale=1.0">
  <title>Wishlist</title>
  <style>
    body {
      font-family: Arial, sans-serif;
      margin: 0;
      padding: 0;
      display: flex;
      flex-direction: column;
      align-items: center;
      justify-content: flex-start;
      min-height: 100vh;
      background-color: #f9f9f9;
      position: relative;
    }

    #background-image {
      position: fixed;
      top: 0;
      left: 0;
      width: 100%;
      height: 100%;
      z-index: -1;
      object-fit: cover;
      opacity: 0.8;
    }

    #content {
      z-index: 1;
      text-align: center;
      margin-top: 20px;
    }

    form {
      margin-bottom: 20px;
    }

    h1 {
      margin-bottom: 20px;
    }

    #wishlists {
      margin-top: 20px;
      width: 90%;
      max-width: 600px;
    }

    #wishlists div {
      border: 1px solid #ddd;
      border-radius: 8px;
      padding: 10px;
      margin-bottom: 10px;
      background: #ffffffdd;
    }
  </style>
</head>
<body>
  <!-- Background Image -->
  <img id="background-image" src="https://i.ibb.co/PtSRCMS/aa.jpg" alt="Christmas Background">

  <!-- Main Content -->
  <div id="content">
    <h1>Wishlist</h1>
    <form id="wishlist-form">
      <input type="text" id="name" placeholder="Your Name" required />
      <textarea id="items" placeholder="Your Wishlist Items (comma-separated)" required></textarea>
      <button type="submit">Submit</button>
    </form>
    <div id="wishlists"></div>
  </div>

  <script>
    const form = document.getElementById('wishlist-form');
    const wishlistsDiv = document.getElementById('wishlists');

    form.addEventListener('submit', async (e) => {
      e.preventDefault();
      const name = document.getElementById('name').value;
      const items = document.getElementById('items').value.split(',');

      await fetch('/api/wishlists', {
        method: 'POST',
        headers: { 'Content-Type': 'application/json' },
        body: JSON.stringify({ name, items }),
      });

      loadWishlists();
    });

    async function loadWishlists() {
      const res = await fetch('/api/wishlists');
      const wishlists = await res.json();
      wishlistsDiv.innerHTML = wishlists
        .map(wishlist => `
          <div>
            <h3>${wishlist.name}</h3>
            <p>${wishlist.items.join(', ')}</p>
          </div>
        `)
        .join('');
    }

    loadWishlists();
  </script>
</body>
</html>
```

![index.html](https://i.imgur.com/i5KE2pR.png)

`src/lib.rs` 에 GET, POST를 처리하기 위한 로직을 구현합니다.
```rust
use serde::{Deserialize, Serialize};
use spin_sdk::http::Method;
use spin_sdk::http::{IntoResponse, Request, Response};
use spin_sdk::http_component;
use spin_sdk::key_value::Store;

#[derive(Debug, Serialize, Deserialize)]
struct Wishlist {
    name: String,
    items: Vec<String>,
}

#[http_component]
fn handle_request(req: Request) -> anyhow::Result<impl IntoResponse> {
    println!("Received request: {:?}", req.method());
    let store = Store::open_default()?;

    let (status, body) = match *req.method() {
        Method::Get => {
            let keys = store.get_keys()?;
            let mut wishlists = vec![];
            for key in keys {
                if let Some(value) = store.get(&key)? {
                    let items: Vec<String> = serde_json::from_slice(&value)?;
                    wishlists.push(Wishlist { name: key, items });
                }
            }
            let json_body = serde_json::to_vec(&wishlists)?;
            (200, json_body)
        }
        Method::Post => {
            let body = req.body();
            let wishlist: Wishlist = match serde_json::from_slice(&body) {
                Ok(wishlist) => wishlist,
                Err(_) => return Ok(Response::builder().status(400).body(Vec::new()).build()),
            };
            let serialized_items = serde_json::to_vec(&wishlist.items)?;
            let _ = store.set(&wishlist.name, &serialized_items);
            (201, Vec::new())
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
    
    ![spin build](https://i.imgur.com/KaMpNai.png)
    
- `spin up` 으로 로컬에서 실행이 가능합니다.
    
    ![spin up](https://i.imgur.com/ZL0h36o.png)
    
- curl을 사용하여 가볍게 테스트가 가능합니다.
    - POST
        
        ```bash
        curl -X POST -H "Content-Type: application/json" \
        -d '{
            "name": "John Doe",  
            "items": ["Ugly Sweater", "Gingerbread House", "Stanley Cup Winter Edition"]
            }' \
        http://localhost:3000/api/wishlists
        ```
        
        ![POST](https://i.imgur.com/RONkzZg.png)
        
    - GET
        
        ```bash
        curl -X GET curl http://127.0.0.1:3000/api/wishlists -H 'Content-Type: application/json'
        ```
        
        ![GET](https://i.imgur.com/H9lE81O.png)
        
- 아래 명령어로 제출 전 테스트가 가능합니다.
    
    ```bash
    hurl --test test.hurl
    ```
    
    ![test](https://i.imgur.com/GcP4bmw.png)