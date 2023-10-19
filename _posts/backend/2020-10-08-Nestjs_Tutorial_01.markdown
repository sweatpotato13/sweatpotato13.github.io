---
layout: post
title: Nestjs 튜토리얼 따라하기 1편
date: "2020-10-08 20:12:58 +0900"
tags: [backend]
---

# Nestjs 튜토리얼 따라하기 1편

https://docs.nestjs.com/

Nestjs는 Node.js의 server-side를 구축하기 위한 프레임워크이다.
<br/>
<br/>

## 설치

Node.js (≥ 10.13.0) 를 필요로 한다.
```bash
$ brew install node
```

시작을 위해 Nest CLI를 설치한다.
```bash
$ npm i -g @nestjs/cli
$ nest new project-name
```

위 명령어를 이용하여 프로젝트를 생성하면 다음과 같은 구조를 가진다.

```text
├── src
│   ├── app.controller.ts
│   ├── app.module.ts
│   └── main.ts
```

|File|Description|
|----|---|
|app.controller.ts|단일 Route를 가진 샘플 컨트롤러|
|app.module.ts|어플리케이션의 Root 모듈|
|main.ts|NestFactory를 사용하여 Nest 어플리케이션 인스턴스를 생성|

<br/>
<br/>

main.ts는 async 함수를 포함하고 있으며 Nest 어플리케이션 인스턴스를 생성한다.

```typescript
import { NestFactory } from '@nestjs/core';
import { AppModule } from './app.module';

async function bootstrap() {
  const app = await NestFactory.create(AppModule);
  await app.listen(3000);
}
bootstrap();
```
<br/>

Nest 어플리케이션 인스턴스를 생성하기 위해서 NestFactory 클래스를 사용한다.
<br/>

NestFactory의 create() 메소드를 사용하게 되면 INestApplication 인터페이스로 구성된 어플리케이션 객체를 반환한다.
<br/>
<br/>

## 실행
```bash
$ npm run start
```
<br/>

어플리케이션을 시작한뒤 http://localhost:3000/ 를 실행하면 Hello World! 메시지를 볼 수 있다.
