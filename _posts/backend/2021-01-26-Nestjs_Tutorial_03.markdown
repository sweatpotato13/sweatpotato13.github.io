---
layout: post
title: Nestjs 튜토리얼 따라하기 3편
date: "2021-01-26 22:31:35 +0900"
tags: [backend]
---

# Nestjs 튜토리얼 따라하기 3편

## Providers

![Providers](https://docs.nestjs.com/assets/Components_1.png)

* Provider는 종속성을 주입하는 기능을 수행함
* ```@Injectable()``` 데코레이터를 사용

```bash
# scr폴더 내에 cats폴더 생성
nest g service cats
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
│       └── cats.service.ts
```
<br/>
<br/>
cats.service.ts에 다음과 같은 코드를 입력한다.

```typescript
// cats.service.ts
import { Injectable } from '@nestjs/common';
import { Cat } from './interfaces/cat.interface';

@Injectable()
export class CatsService {
  private readonly cats: Cat[] = [];

  create(cat: Cat) {
    this.cats.push(cat);
  }

  findAll(): Cat[] {
    return this.cats;
  }
}
```

이를 사용하기 위해서 CatsController에 연결해준다.
```typescript
// cats.controller.ts
import { Controller, Get, Post, Body } from '@nestjs/common';
import { CreateCatDto } from './dto/create-cat.dto';
import { CatsService } from './cats.service';
import { Cat } from './interfaces/cat.interface';

@Controller('cats')
export class CatsController {
  constructor(private catsService: CatsService) {}

  @Post()
  async create(@Body() createCatDto: CreateCatDto) {
    this.catsService.create(createCatDto);
  }

  @Get()
  async findAll(): Promise<Cat[]> {
    return this.catsService.findAll();
  }
}
```

생성자를 통하여 catsService를 선언하고 catsService에 선언된 함수를 사용하였다.

<br/>
<br/>

CatsService, CatsController를 정의하였으므로 이를 수행할 수 있도록 module에 등록하여야 한다.
```typescript
// app.module.ts
import { Module } from '@nestjs/common';
import { CatsController } from './cats/cats.controller';
import { CatsService } from './cats/cats.service';

@Module({
  controllers: [CatsController],
  providers: [CatsService],
})
export class AppModule {}
```
