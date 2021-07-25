---
layout: post
title: Nest.js 튜토리얼 따라하기 5편
date: "2021-01-26 22:31:35 +0900"
tags: [nest.js]
---

# Nest.js 튜토리얼 따라하기 5편

## Middleware

![Middleware](https://docs.nestjs.com/assets/Middlewares_1.png)

* Middleware는 요청 및 응답에 엑세스 가능

Nest의 Middleware는 Express의 Middleware와 동일하다.
Express의 공식 문서에서 설명하는 미들웨어의 기능은 아래와 같다.
<br/>
```text
execute any code.

make changes to the request and the response objects.

end the request-response cycle.


call the next middleware function in the stack.

if the current middleware function does not end the request-response cycle, it must call next() to pass control to the next middleware function. 

Otherwise, the request will be left hanging.
```
<br/>

Nest의 Middleware는 함수나 ```@Injectable()``` 데코레이터가 있는 클래스에서 구현 된다.

```typescript
// logger.middleware.ts
import { Injectable, NestMiddleware } from '@nestjs/common';
import { Request, Response, NextFunction } from 'express';

@Injectable()
export class LoggerMiddleware implements NestMiddleware {
  use(req: Request, res: Response, next: NextFunction) {
    console.log('Request...');
    next();
  }
}
```

또한 Nest의 Middleware는 모듈 내에서 사용할 수 있는 종속성을 생성자를 이용하여 삽입할 수 있다.
<br>
<br>

Nest의 Middleware 적용을 위해서 Module클래스 내에서 ```configure()```를 사용하여야 한다. 또한 Middleware를 포함하는 모듈은 ```NestModule``` 인터페이스를 ```implements NestModule ``` 형식으로 사용하여야 한다.
```typescript
import { Module, NestModule, MiddlewareConsumer } from '@nestjs/common';
import { LoggerMiddleware } from './common/middleware/logger.middleware';
import { CatsModule } from './cats/cats.module';

@Module({
  imports: [CatsModule],
})
export class AppModule implements NestModule {
  configure(consumer: MiddlewareConsumer) {
    consumer
      .apply(LoggerMiddleware)
      .forRoutes('cats');
  }
}
```
<br>

또한 Middleware를 특정 Method로 제한할 수 있다.
```typescript
import { Module, NestModule, RequestMethod, MiddlewareConsumer } from '@nestjs/common';
import { LoggerMiddleware } from './common/middleware/logger.middleware';
import { CatsModule } from './cats/cats.module';

@Module({
  imports: [CatsModule],
})
export class AppModule implements NestModule {
  configure(consumer: MiddlewareConsumer) {
    consumer
      .apply(LoggerMiddleware)
      .forRoutes({ path: 'cats', method: RequestMethod.GET });
  }
}
```
<br>

와일드카드도 적용이 가능하다.
```typescript
import { Module, NestModule, RequestMethod, MiddlewareConsumer } from '@nestjs/common';
import { LoggerMiddleware } from './common/middleware/logger.middleware';
import { CatsModule } from './cats/cats.module';

@Module({
  imports: [CatsModule],
})
export class AppModule implements NestModule {
  configure(consumer: MiddlewareConsumer) {
    consumer
      .apply(LoggerMiddleware)
      forRoutes({ path: 'ab*cd', method: RequestMethod.ALL });
  }
}
```
<br>

Middleware 또한 전역으로 선언할 수 있다
```typescript
const app = await NestFactory.create(AppModule);
app.use(logger);
await app.listen(3000);
```