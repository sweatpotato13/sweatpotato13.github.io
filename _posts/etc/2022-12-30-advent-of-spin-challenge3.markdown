---
layout: post
title: "Advent-of-spin Challenge 3"
date: "2022-12-30 08:05:44 +0900"
tags: [etc]
---

이 포스트는 [advent-of-spin](https://github.com/fermyon/advent-of-spin)에 업로드 된 Challenge 3에 대해서 진행했던 내용을 정리한 포스트 입니다.

Rust 언어로 진행하였으며 이 포스트에 게시된 소스코드는 [Github](https://github.com/sweatpotato13/advent-of-spin/tree/main/2023/CHALLENGE-3) 에서 확인할 수 있습니다.

## Spec

- `/` route에서 HTTP 200을 응답으로 리턴
  - response header에 `Content-Type: text/html`
  - `<h1>` 태그에 “Welcome to the Advent of Spin!” 메시지가 페이지에 포함되어야 함
  - Body에 “Santa’s on his way” 가 포함되어야 함

## Work

- https://github.com/fermyon/bartholomew-site-template.git 를 기반으로 프로젝트를 구성합니다.
- `spin up` 으로 프로젝트 실행 후 `curl -v [http://localhost:3000](http://localhost:3000)` 으로 접속해보면 아래와 같은 로그를 통하여 헤더에 `text/html` 이 포함되어 있음을 확인할 수 있다.
  ![Untitled](https://i.imgur.com/8NmJfm3.png)
- `content/index.md` 를 아래와 같이 수정하였습니다.
  ![Untitled](https://i.imgur.com/OWvOr2T.png)
- `spin deploy` 로 클라우드에 업로드
  ![Untitled](https://i.imgur.com/YBwGyH5.png)
- 위에서 획득한 URl을 통해 제출
  ```rust
  hurl --variable serviceUrl="https://bartholomew-template-oskncmgc.fermyon.app" submit.hurl
  ```
  ![Untitled](https://i.imgur.com/zEgX4zG.png)

---
