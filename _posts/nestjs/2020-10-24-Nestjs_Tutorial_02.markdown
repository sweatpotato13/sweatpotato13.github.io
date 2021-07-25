---
layout: post
title: Nest.js 튜토리얼 따라하기 2편
date: "2020-10-24 20:12:58 +0900"
tags: [nest.js]
---

# Nest.js 튜토리얼 따라하기 2편

## Controllers

![controllers](https://i.imgur.com/KGC7Dm3.png)

* Controller는 서버 어플리케이션을 위하여 특정 요청을 받아들이는 역할을 한다.
* Controller를 만들기 위하여 클래스와 데코레이터를 사용
* 데코레이터는 요청을 해당 Controller에 연결한다.

```bash
# scr폴더 내에 cats폴더 생성
nest g controller cats
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
│       └── cats.controller.ts
```
<br/>
<br/>
cats.controller.ts에 다음과 같은 코드를 입력한다.

```typescript
// src/cats/cats.controller.ts
import { Controller, Get } from '@nestjs/common';

@Controller('cats')
export class CatsController {
    @Get()
    findAll() : string {
        return 'This action returns all cats';
    }
}
```

이후 http://localhost:3000/cats/ 에 접속

![cats](https://i.imgur.com/IlmRTyj.png)

@Controller('cats') 데코레이터로 인하여 CatsController에 연결되었고 GET method로 연결하였기 때문에 @Get() 데코레이터가 붙어있는 findAll() 함수가 실행된 모습을 볼 수 있다.

<br/>
<br/>
cats.controller.ts를 다음과 같이 수정한다.

```typescript
// src/cats/cats.controller.ts
import { Controller, Get, Param, Req } from '@nestjs/common';
import { Request } from 'express';

@Controller('cats')
export class CatsController {
  @Get()
  findAll(@Req() request: Request): string {
    return 'This action returns all cats';
  }
  @Get(':id')
  find(@Param('id')id) {
      return id;
  }
}
```

이후 http://localhost:3000/cats/1 에 접속

![cats](https://i.imgur.com/uzuJhWf.png)

위의 find()함수와 같이 사용하면 파라미터도 같이 넘길 수 있다.

<br/>
<br/>
cats.controller.ts를 다음과 같이 수정한다.

```typescript
// src/cats/cats.controller.ts
import { Controller, Get, HttpCode, Param, Post, Req } from '@nestjs/common';
import { Request } from 'express';

@Controller('cats')
export class CatsController {
    @Post()
    @HttpCode(200)
    create(): string {
        return 'This action adds a new cat';
    }

    @Get()
    findAll(@Req() request: Request): string {
        return 'This action returns all cats';
    }
    @Get(':id')
    find(@Param('id') id) {
        return id;
    }
}
```

POST method로 request를 전송해보자.

![cats](https://i.imgur.com/k0jOqyg.png)

정상적으로 create() 함수가 실행되었다.

<br/>
<br/>
cats.controller.ts를 다음과 같이 수정한다.

```typescript
// src/cats/cats.controller.ts
import { identifier } from '@babel/types';
import { Body, Controller, Get, Param, Post, Put, Req } from '@nestjs/common';
import { Request } from 'express';

@Controller('cats')
export class CatsController {
    @Post()
    create(@Body('name') name) {
        return { name };
    }
    @Put()
    update(@Body('id') id, @Body('name') name) {
        return { id, name };
    }
    @Get()
    findAll(@Req() request: Request): string {
        return 'This action returns all cats';
    }
    @Get(':id')
    find(@Param('id') id) {
        return id;
    }   
}
```

![cats](https://i.imgur.com/1OIhpU8.png)

PUT method로 id 와 name을 파라미터로 하여 request를 날리고 그 결과 정상적으로 update()함수가 실행 되었다.
