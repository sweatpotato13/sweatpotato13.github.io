---
layout: post
title: "Express vs Fastify 서버 프레임워크 비교"
date: "2021-11-14 13:03:44 +0900"
tags: [etc]
---

## Express vs Fastify

저는 백엔드 프레임워크로 Nest.js를 사용하고 있습니다. Nest.js 는 Express 혹은 Fastify 기반으로 작성을 할 수 있기 때문에 자연스럽게 Express와 Fastify를 비교하여 어떤 경우에 어떤 것을 취해서 개발을 해야할지 고민이 되었습니다. 이같은 문제를 해결하기 위해선 이 두 프레임워크의 비교가 불가피 하였습니다.

두 프레임워크 모두 기본적으로 TypeScript 대응, 라우터 기능, Validator 기능 등 기본적인 서버 프레임워크가 갖춰야할 사항은 모두 갖추었는데 어느 부분에서 차이가 나는 것일까요?

### Express

![Untitled](https://i.imgur.com/IRFhfsT.png)

Express는 이전부터 현재까지 쭉 Node.js의 표준처럼 사용 되어진 서버 프레임 워크입니다. 대부분 Node.js 기반으로 서버를 개발한다고 하면 Express를 떠올릴만큼 대중적이며 이 같은 이유로 참고할 자료도 훨씬 방대하고 개발자들도 익숙한 프레임워크 입니다. Fastify와 다르게 비동기 처리를 지원하지 않습니다. 

### Fastify

![Untitled](https://i.imgur.com/TyJv9DP.png)

Fastify는 이름처럼 Express보다 빠른 성능을 강조하는 프레임워크 입니다. Fastify측에서 제공하는 [벤치마크](https://www.fastify.io/benchmarks/)에 의하면 간단한 hello world! 오버헤드를 측정하는데 Express보다 월등하게 뛰어난 성능을 보여주고 있습니다. 

### Benchmark

그러면 실제로 fastify와 express의 성능 벤치마크를 확인하여 성능차이를 확인해 봅시다.

[https://github.com/foxifyjs/fastify-benchmarks](https://github.com/foxifyjs/fastify-benchmarks)

![Untitled](https://i.imgur.com/Cm30TR4.png)

express와 fastify만 비교해서 살펴본다면 약 2배 정도의 요청을 처리하는 것으로 결과가 나왔습니다.