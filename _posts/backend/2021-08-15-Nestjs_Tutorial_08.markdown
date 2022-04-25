---
layout: post
title: Nest.js 튜토리얼 따라하기 8편
date: "2021-08-15 22:31:45 +0900"
tags: [backend]
---

# Nest.js 튜토리얼 따라하기 8편

## Guards

![Guards](https://docs.nestjs.com/assets/Guards_1.png)

* Guard는 @Injectable() 데코레이터를 사용하는 클래스이며 `CanActivate` 인터페이스를 구현하여 정의 된다.

* Guard는 권한부여에 대한 결과를 처리한다. 이는 일반적으로 express의 미들웨어에 의해 처리 되었다.

express의 미들웨어는 next()를 호출하게 되면 그 이후에 무슨 함수가 호출될 지 모르지만 Guards는 `ExecutionContext`에 접근할 수 있으므로 다음에 실행될 항목을 정확하게 알 수 있다.

### Authorization guard

```typescript
import { Injectable, CanActivate, ExecutionContext } from '@nestjs/common';
import { Observable } from 'rxjs';

@Injectable()
export class AuthGuard implements CanActivate {
  canActivate(
    context: ExecutionContext,
  ): boolean | Promise<boolean> | Observable<boolean> {
    const request = context.switchToHttp().getRequest();
    return validateRequest(request);
  }
}
```

### Execution context

```canActive()` 함수는 파라미터로 `ExecutionContext` 를 갖는다. `ExecutionContext`는 `ArgumentHost`로 부터 상속된다. 

### Role-based authentication

```typescript
import { Injectable, CanActivate, ExecutionContext } from '@nestjs/common';
import { Observable } from 'rxjs';

@Injectable()
export class RolesGuard implements CanActivate {
  canActivate(
    context: ExecutionContext,
  ): boolean | Promise<boolean> | Observable<boolean> {
    return true;
  }
}
```

### Guard 적용하기

Pipe처럼 Guard도 Controller에 적용하거나, 전역으로 적용할 수 있다. Controller에 적용하기 위해서 `@UseGuards()` 데코레이터를 사용해야 한다.

```typescript
@Controller('cats')
@UseGuards(RolesGuard)
export class CatsController {}
```

전역으로 적용하기 위해서 `useGlobalGuards()` 메소드를 사용한다.
```typescript
const app = await NestFactory.create(AppModule);
app.useGlobalGuards(new RolesGuard());
```

```typeScript
import { Module } from '@nestjs/common';
import { APP_GUARD } from '@nestjs/core';

@Module({
  providers: [
    {
      provide: APP_GUARD,
      useClass: RolesGuard,
    },
  ],
})
export class AppModule {}
```

### Setting roles per handler

위에서 구현한 `RolesGuard`가 잘 작동하기 위해서 각 Controller Route마다 어떤 권한을 가지고 있을 때 권한을 허가할건지 설정해야한다. 
```typeScript
@Post()
@SetMetadata('roles', ['admin'])
async create(@Body() createCatDto: CreateCatDto) {
  this.catsService.create(createCatDto);
}
```

혹은 직접 데코레이터를 생성해서 권한을 설정할 수 있다.
```typescript
import { SetMetadata } from '@nestjs/common';

export const Roles = (...roles: string[]) => SetMetadata('roles', roles);
```

```typeScript
@Post()
@Roles('admin')
async create(@Body() createCatDto: CreateCatDto) {
  this.catsService.create(createCatDto);
}
```

위에서 구현한 것을 모두 합치면
```typescript
import { Injectable, CanActivate, ExecutionContext } from '@nestjs/common';
import { Reflector } from '@nestjs/core';

@Injectable()
export class RolesGuard implements CanActivate {
  constructor(private reflector: Reflector) {}

  canActivate(context: ExecutionContext): boolean {
    const roles = this.reflector.get<string[]>('roles', context.getHandler());
    if (!roles) {
      return true;
    }
    const request = context.switchToHttp().getRequest();
    const user = request.user;
    return matchRoles(roles, user.roles);
  }
}
```


