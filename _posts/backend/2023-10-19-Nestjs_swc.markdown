---
layout: post
title: "Nestjs에 SWC 컴파일러 적용하기
date: "2023-10-19 13:03:44 +0900"
tags: [backend]
---

본 문서는 [Nestjs 공식문서](https://docs.nestjs.com/recipes/swc) 를 참조하여 작성 되었습니다.

---

## Overview

Nestjs v10부터 [swc](https://swc.rs/) 를 사용할 수 있게 되었습니다. 사실 v9때도 불가능 했던 것은 아니지만 공식 문서에서도

```SWC isn't compatible with NestJS CLI plugins.``` 

이라고 안내를 했었던 만큼 v10에서 공식적인 지원이 되었다 라고 봐도 괜찮을 것 같습니다.

본 문서에서는 제가 사용하고 있는 [boilerplate](https://github.com/sweatpotato13/nestjs-boilerplate)에 어떤 방법으로 swc를 적용하였는지에 대하여 작성하였습니다.

## Installation

시작을 위하여 아래 명령어를 이용하여 패키지를 설치 해 주었습니다.
```bash
yarn add -D @swc/cli @swc/core
```

## Getting Started

위 패키지를 설치하고 nest-cli에 `-b` 옵션을 통해서 swc를 적용할 수 있습니다.
```bash
nest start -b swc
```

위 명령어처럼 `-b` 옵션을 주는것으로 실행이 가능 하지만 `nest-cli.json`에 아래 내용을 추가하는 것으로  builder로 swc를 사용하게 끔 설정할 수 있습니다.

```json
{
  "compilerOptions": {
    "builder": "swc"
  }
}
```

아니면 아래와 같이 추가적인 옵션을 설정하는 것도 가능합니다.
```json
"compilerOptions": {
  "builder": {
    "type": "swc",
    "options": {
      "swcrcPath": "infrastructure/.swcrc"
    }
  }
}
```


제가 실제 사용하고 있는 `nest-cli.json` 내용은 아래와 같습니다.
```json
{
    "language": "ts",
    "collection": "@nestjs/schematics",
    "sourceRoot": "src",
    "compilerOptions": {
        "builder": {
          "type": "swc",
          "options": {
            "swcrcPath": ".swcrc"
          }
        }
      }      
}
```


위 파일에서 `.swcrc` 파일을 사용하고 있는데 swc에 대한 옵션을 구성하는 파일입니다.

기본적으로 루트 디렉토리에 두어야 하지만, 다른곳에 두더라도 `nest-cli.json` 에서 `.swcrc` 파일 위치를 지정할 수 있습니다.

`.swcrc` 파일에 대한 예시는 아래와 같습니다.

```json
{
    "sourceMaps": true,
    "module": {
        "type": "commonjs",
        "strict": true
    },
    "jsc": {
        "target": "es2020",
        "parser": {
            "syntax": "typescript",
            "decorators": true
        },
        "transform": {
            "legacyDecorator": true,
            "decoratorMetadata": true
        },
        "keepClassNames": true
    }
}
```

여기까지 진행 했다면 `nest start` 혹은 `nest build` 등 기존 명령어를 그대로 수행하더라도 swc가 적용되어 실행 되는 모습을 확인할 수 있습니다.

![image](https://i.imgur.com/ESP3Jnj.png)

## TypeORM

만약 TypeORM을 사용하고 있는 경우 relation을 사용하는 경우 아래와 같은 오류를 볼 수 있습니다.

![image2](https://i.imgur.com/zlIDrYu.png)

```ts
@Entity("role", { schema: "public" })
export class Role {

    // ---- snip ----
    
    @OneToMany(() => UserRole, userRole => userRole.role)
    userRoles: UserRole[];

// ---- snip ----
```

위 처럼 relation을 사용하는 경우 아래와 같이 처리를 해 주어야 합니다.
```ts
@Entity("role", { schema: "public" })
export class Role {

    // ---- snip ----
    
    @OneToMany(() => UserRole, userRole => userRole.role)
    userRoles: Relation<UserRole[]>;

// ---- snip ----
```

---

위와 같은 방법으로 Nestjs에 swc를 적용 해 보았습니다. 실제로 사용 해 보면서 확실히 빌드 속도가 빨라졌다 라는 것이 느껴질 정도로 속도에 변화가 있었습니다.

Nestjs를 사용하고 있다면 한번쯤 swc를 사용 해 보는것도 좋은 선택인 것 같습니다.

본 문서에서 사용된 모든 코드는 [boilerplate](https://github.com/sweatpotato13/nestjs-boilerplate) 에 포함되어 있습니다.