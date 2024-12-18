---
layout: post
title: "2024 Advent-of-spin Challenge 3"
date: "2024-12-17 08:05:44 +0900"
tags: [etc]
---

이 포스트는 [advent-of-spin](https://github.com/fermyon/advent-of-spin)에 업로드 된 Challenge 3에 대해서 진행했던 내용을 정리한 포스트 입니다.

Rust 언어로 진행하였으며 이 포스트에 게시된 소스코드는 [Github](https://github.com/sweatpotato13/advent-of-spin/tree/main/2024/CHALLENGE-3) 에서 확인할 수 있습니다.

## Spec

- Gift-Suggestion-Generator Wasm 컴포넌트
- `/gift-suggestions.html` 페이지 추가
    - 사용자 이름, 나이, 취향을 입력하면 선물 제안을 생성하는 API를 호출하는 페이지
- `/api/generate-gift-suggestions` path에서 POST method 구현하기'
   - AI가 선물 제안을 생성하는 API
   - request
       ```json
       {
            "name": "Riley Parker",
            "age": 15,
            "likes": "Computers, Programming, Mechanical Keyboards"
        }
       ```
    - response
        ```json
        {
            "name": "Riley Parker",
            "giftSuggestions": "I bet Riley would be super happy with a new low profile mechanical keyboard or a couple of new books about software engineering"
        }
        ```

## Work

이 Challenge도 Challenge 2과 마찬가지로 static page와 api가 구현된 wasm component를 사용하기 위해 challenge 2에서 만들어놓은 프로젝트를 기반으로 사용하려고 합니다.

우선 프론트엔드 부분을 구현하기 위해 `assets/gift-suggestions.html` 파일을 생성합니다.

`assets/gift-suggestions.html` 경로에 GET 요청을 보내면 아래와 같은 응답을 받을 수 있도록 구현합니다.
        
```html
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Gift Suggestions</title>
    <style>
        body {
            font-family: Arial, sans-serif;
            margin: 20px;
            background-color: #f9f9f9;
        }
        h1 {
            color: #2c3e50;
        }
        form {
            display: flex;
            flex-direction: column;
            gap: 10px;
            max-width: 400px;
            margin-bottom: 20px;
        }
        label {
            font-weight: bold;
        }
        input, button {
            padding: 10px;
            font-size: 1rem;
        }
        button {
            background-color: #3498db;
            color: white;
            border: none;
            cursor: pointer;
        }
        button:hover {
            background-color: #2980b9;
        }
        #result {
            background-color: white;
            border: 1px solid #ddd;
            padding: 15px;
            max-width: 400px;
        }
    </style>
</head>
<body>
    <h1>🎁 Gift Suggestions</h1>
    <form id="giftForm">
        <label for="name">Name:</label>
        <input type="text" id="name" placeholder="Enter name" required>

        <label for="age">Age:</label>
        <input type="number" id="age" placeholder="Enter age" required>

        <label for="likes">Likes/Interests:</label>
        <input type="text" id="likes" placeholder="e.g., Programming, Sports" required>

        <button type="submit">Get Gift Suggestions</button>
    </form>

    <div id="result" hidden>
        <h2>Gift Suggestions for <span id="childName"></span></h2>
        <p id="suggestions"></p>
    </div>

    <script>
        const form = document.getElementById("giftForm");
        const resultDiv = document.getElementById("result");
        const childNameSpan = document.getElementById("childName");
        const suggestionsParagraph = document.getElementById("suggestions");

        form.addEventListener("submit", async (event) => {
            event.preventDefault();

            // 폼에서 데이터 가져오기
            const name = document.getElementById("name").value;
            const age = parseInt(document.getElementById("age").value);
            const likes = document.getElementById("likes").value;

            // API 요청
            try {
                const response = await fetch("/api/generate-gift-suggestions", {
                    method: "POST",
                    headers: { "Content-Type": "application/json" },
                    body: JSON.stringify({ name, age, likes })
                });

                if (!response.ok) {
                    throw new Error(`API Error: ${response.status}`);
                }

                const data = await response.json();

                // 결과 표시
                childNameSpan.textContent = name;
                suggestionsParagraph.textContent = data.giftSuggestions;
                resultDiv.hidden = false;
            } catch (error) {
                console.error("Error fetching gift suggestions:", error);
                alert("Failed to fetch gift suggestions. Please try again later.");
            }
        });
    </script>
</body>
</html>
```

![code](https://i.imgur.com/8eOyJ9D.png)

![index.html](https://i.imgur.com/dQwabNa.png)

이번에도 `Component Dependencies` 기능을 사용하여 선물 아이디어를 반환하도록 하는 기능이 포함된 wasm component를 추가해보겠습니다.

이 컴포넌트는 [gIthub](https://github.com/fermyon/advent-of-spin/blob/main/2024/Challenge-3) 의 `./template` 폴더에 포함되어 있으므로 해당 파일을 가져와서 사용하겠습니다.

![ss](https://i.imgur.com/B0v2kK4.png)

첫번째로 venv를 사용하여 가상 환경을 구성합니다.

```bash
python3 -m venv venv
```

그러면 `venv` 폴더가 생성됩니다.

![ss](https://i.imgur.com/3DgYZhy.png)

이제 가상 환경을 활성화합니다.

```bash
source venv/bin/activate
```

![ss](https://i.imgur.com/0WHcN03.png)

이후에 Dependencies를 설치합니다.

```bash
pip install -r requirements.txt
```

![ss](https://i.imgur.com/LF6GAZd.png)

이제 wasm component를 구현 해야합니다. `app.py` 파일을 확인하면 아래와 같은 코드를 확인할 수 있습니다.

```python
from gift_suggestions_generator import exports
from gift_suggestions_generator.exports.generator import Suggestions
# from spin_sdk import llm

class Generator(exports.Generator):
    def suggest(self, name: str, age: int, likes: str):
        # Implement your gift suggestion here
        return Suggestions("John Doe", "I bet John would be super happy with a new mechanical keyboard.")
```

우리는 이 코드를 llm을 이용하여 더 적절하게 추천을 할 수 있도록 수정해보겠습니다.

```python
from gift_suggestions_generator import exports
from gift_suggestions_generator.exports.generator import Suggestions
from spin_sdk import llm

class Generator(exports.Generator):
    def suggest(self, name: str, age: int, likes: str):
        prompt = (
            f"Suggest a personalized gift for {name}, "
            f"a {age}-year-old person who likes {likes}. "
            "Be creative and thoughtful, and provide a brief reason for your suggestion."
        )
        
        try:
            result = llm.infer("llama2-chat", prompt)
            suggestion_text = result.text  
        except Exception as e:
            suggestion_text = "Unable to generate a suggestion at the moment. Please try again later."

        return Suggestions(name, suggestion_text)
```

이제 wasm component를 생성하기 위해 아래 커맨드를 입력합니다.

```bash
componentize-py -d ./wit/ -w gift-suggestions-generator componentize -m spin_sdk=spin-imports app -o gift-suggestions-generator.wasm
```

![ss](https://i.imgur.com/nVCSM26.png)

그러면 `gift-suggestions-generator.wasm` 파일이 생성됩니다.

![ss](https://i.imgur.com/MzKAtIm.png)

이제 이 wasm 파일을 spin 프로젝트에 추가해야 합니다.

`spin deps add ./gift-suggestions-generator.wasm` 커맨드를 사용하여 spin 프로젝트에 추가해 줍니다.

![ss](https://i.imgur.com/XhGvnE0.png)

![ss](https://i.imgur.com/VaBx0Rp.png)

이후 `spin deps generate-bindings` 커맨드를 사용하여 wasm 파일에 대한 binding을 생성해 주어야 하는데 이를 위해서 spin plugin을 설치해 주어야 합니다.

- https://github.com/fermyon/spin-deps-plugin/tree/main?tab=readme-ov-file#installation

설치가 완료 되었다면 아래 명령어를 사용하여 binding을 생성합니다.

```bash
spin deps generate-bindings -L rust -o src/bindings -c challenge3
```

위 과정이 정상적으로 수행 되엇다면 `src/bindings` 경로에 파일이 생성된 것을 확인할 수 있습니다.

![ss](https://i.imgur.com/kz8SDWq.png)

![ss](https://i.imgur.com/1a55EVl.png)

이제 이 파일을 이용하여 rust 코드를 작성해보겠습니다.

`src/lib.rs` 에 GET을 처리하기 위한 로직을 구현합니다.
```rust
use serde::{Deserialize, Serialize};
use spin_sdk::http::Method;
use spin_sdk::http::{IntoResponse, Request, Response};
use spin_sdk::http_component;

mod bindings;

#[derive(Debug, Serialize, Deserialize)]
struct SuggestionsRequest {
    name: String,
    age: u8,
    likes: String,
}

#[derive(Debug, Serialize, Deserialize)]
struct SuggestionsResonse {
    name: String,
    #[warn(non_snake_case)]
    giftSuggestions: String,
}

#[http_component]
fn handle_request(req: Request) -> anyhow::Result<impl IntoResponse> {
    println!("Received request: {:?}", req.method());

    let (status, body) = match *req.method() {
        Method::Post => {
            let body = req.body();
            let suggestions_request: SuggestionsRequest = match serde_json::from_slice(&body) {
                Ok(suggestions_request) => suggestions_request,
                Err(_) => return Ok(Response::builder().status(400).body(Vec::new()).build()),
            };
            let gift_suggestions = bindings::deps::components::advent_of_spin::generator::suggest(
                &suggestions_request.name,
                suggestions_request.age,
                &suggestions_request.likes,
            )
            .unwrap();
            let json_body = SuggestionsResonse {
                name: gift_suggestions.clone().name,
                giftSuggestions: gift_suggestions.clone().suggestions,
            };
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

실행 전 `spin.toml` 에 해당 옵션을 추가해 줍니다.
```toml
dependencies_inherit_configuration = true
ai_models = ["llama2-chat"]
```

![ss](https://i.imgur.com/jhMWOxh.png)


- `spin build` 로 빌드를 수행합니다.
    
    ![spin build](https://i.imgur.com/4DIcDKb.png)
    
- `spin up` 으로 로컬에서 실행이 가능합니다.
    
    ![spin up](https://i.imgur.com/6qxAQLS.png)
    
- curl을 사용하여 가볍게 테스트가 가능합니다.
    - POST
        ```bash
            curl -X POST \             
            -H "Content-Type: application/json" \
            -d '{
                "name": "John Doe",
                "age": 15,
                    "likes": "Computers, Programming, Mechanical Keyboards"
                }' \
            http://localhost:3000/api/generate-gift-suggestions
        ```
        
- 아래 명령어로 제출 전 테스트가 가능합니다.
    
    ```bash
    hurl --test test.hurl
    ```
    
    ![test](https://i.imgur.com/DAjoQL3.png)