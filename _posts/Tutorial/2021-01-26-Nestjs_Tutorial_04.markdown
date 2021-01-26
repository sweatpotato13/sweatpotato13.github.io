---
layout: post
title: Nest.js 튜토리얼 따라하기 4편
date: "2021-01-26 22:31:35 +0900"
categories: [Tutorial]
tags: [nestjs,tutorial]
---

# Nest.js 튜토리얼 따라하기 4편

## Modules

![Modules](https://docs.nestjs.com/assets/Modules_1.png)

* Module은 어플리케이션의 구조를 구성하는데 사용
* ```@Module()``` 데코레이터를 사용

```bash
# scr폴더 내에 cats폴더 생성
nest g module cats
```
<br/>
<br/>
위 명령어를 이용하여 프로젝트를 생성하면 다음과 같은 구조를 가진다.

```text
├── src
│   ├── app.controller.ts
│   ├── app.module.ts
│   ├── main.ts
│   └── cats
│       └── cats.module.ts
```
<br/>
<br/>
cats.module.ts에 다음과 같은 코드를 입력한다.

```typescript
// cats.module.ts
import { Module } from '@nestjs/common';
import { CatsController } from './cats.controller';
import { CatsService } from './cats.service';

@Module({
  controllers: [CatsController],
  providers: [CatsService],
})
export class CatsModule {}
```

<br>
또한 여기서 생성된 CatsModule을 루트 모듈로 연결시켜 준다.

```typescript
// app.module.ts
import { Module } from '@nestjs/common';
import { CatsModule } from './cats/cats.module';

@Module({
  imports: [CatsModule],
})
export class AppModule {}
```
이제 루트 모듈은 CatsModule에 접근할 수 있게된다.

<br>
<br>

### Global Module

모든 곳에서 동일한 모듈을 가져오기 위해서 모든 module.ts에 동일한 선언을 한다면 그것은 비효율적일 것이다. 이 같은 경우를 위해서 ```@Global()```을 이용하여 Module을 전역으로 설정할 수 있다.

```typescript
import { Module, Global } from '@nestjs/common';
import { CatsController } from './cats.controller';
import { CatsService } from './cats.service';

@Global()
@Module({
  controllers: [CatsController],
  providers: [CatsService],
  exports: [CatsService],
})
export class CatsModule {}
```
전역으로 선언된 모듈은 루트 혹은 코어 모듈에서 한번만 등록해야 한다.