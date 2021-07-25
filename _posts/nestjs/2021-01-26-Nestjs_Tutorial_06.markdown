---
layout: post
title: Nest.js 튜토리얼 따라하기 6편
date: "2021-01-26 22:31:35 +0900"
tags: [nest.js]
---

# Nest.js 튜토리얼 따라하기 6편

## Exception filters

![Exception filters](https://docs.nestjs.com/assets/Filter_1.png)

* Nest의 모든 예외를 처리하는 Exception filters가 존재

Nest는 기본적으로 ```HttpException```이 내장되어 있으며 이는 ```@nestjs/common``` 패키지에서 불러올 수 있다.
```typescript
@Get()
async findAll() {
  throw new HttpException('Forbidden', HttpStatus.FORBIDDEN);
}
```
<br>

```HttpException```은 2개의 인수로 구성된다.
* ```response``` : response를 정의한다. 이는 object일 수 있으며 string일 수도 있다.
* ```status``` : HTTP 상태 코드를 정의한다.

```response```의 JSON object는 2가지 속성이 필요하다.
* ```statusCode``` : 기본값은 status 값에 작성된 값
* ```message``` : HTTP 오류에 대한 설명

```typescript
@Get()
async findAll() {
  throw new HttpException({
    status: HttpStatus.FORBIDDEN,
    error: 'This is a custom message',
  }, HttpStatus.FORBIDDEN);
}
```
```json
{
  "status": 403,
  "error": "This is a custom message"
}
```

### Exception filters
```typescript
import { ExceptionFilter, Catch, ArgumentsHost, HttpException } from '@nestjs/common';
import { Request, Response } from 'express';

@Catch(HttpException)
export class HttpExceptionFilter implements ExceptionFilter {
  catch(exception: HttpException, host: ArgumentsHost) {
    const ctx = host.switchToHttp();
    const response = ctx.getResponse<Response>();
    const request = ctx.getRequest<Request>();
    const status = exception.getStatus();

    response
      .status(status)
      .json({
        statusCode: status,
        timestamp: new Date().toISOString(),
        path: request.url,
      });
  }
}
```
<br>

모든 Exception filters는 ```implements ExceptionFilter```를 포함하여야 한다. 

위에서 구현한 ```HttpExceptionFilter```를 적용하는 방법은 아래와 같다.
```typescript
@Post()
@UseFilters(new HttpExceptionFilter())
async create(@Body() createCatDto: CreateCatDto) {
  throw new ForbiddenException();
}
```
여기서 ```@UseFilters()```는 ```@nestjs/common``` 패키지에서 제공된다.

<br>

또한 필터를 전역으로 적용하기 위하여 아래와 같이 코드를 작성한다.
```typescript
async function bootstrap() {
  const app = await NestFactory.create(AppModule);
  app.useGlobalFilters(new HttpExceptionFilter());
  await app.listen(3000);
}
bootstrap();
```

<br>

혹은 모듈 단위로 필터를 적용할 수 있다.
```typescript
import { Module } from '@nestjs/common';
import { APP_FILTER } from '@nestjs/core';

@Module({
  providers: [
    {
      provide: APP_FILTER,
      useClass: HttpExceptionFilter,
    },
  ],
})
export class AppModule {}
```